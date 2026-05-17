# Layer 0 — 인프라 셋업 (Week 1)

10개 자동화 모두가 공유하는 기반. 이걸 끝내야 어떤 자동화도 시작할 수 있다.

**완료 기준**: 빈 n8n 워크플로 하나가 매일 정해진 시간에 Slack `#automation-log`로 "hello"를 보낸다.

**예상 소요**: 풀타임 1~2일

---

## 0. 체크리스트 (전체)

- [ ] **1.** n8n 인스턴스 (Docker/Railway/cloud)
- [ ] **2.** Notion 워크스페이스 + DB 6종 (`ContentIdeas`, `ContentDrafts`, `Leads`, `CampaignMetrics`, `Inbox`, `CompetitorTimeline`)
- [ ] **3.** Slack 워크스페이스 + 5개 채널 + 봇 토큰
- [ ] **4.** API 키 발급 (Anthropic, OpenAI 선택)
- [ ] **5.** n8n credentials 등록 (Notion / Slack / Anthropic)
- [ ] **6.** `.env` 파일 + 환경변수 주입
- [ ] **7.** Hello World 워크플로 임포트·실행 성공
- [ ] **8.** 자동화 활성화 → 매일 Slack 메시지 도착 확인

---

## 1. n8n 셋업

선택지 3개. **빠르게 시작 → Railway. 비용 최소 → Hetzner. 학습 안 함 → n8n Cloud.**

### Option A: Railway (권장, 5분)

1. https://railway.app → 가입 (GitHub OAuth)
2. New Project → Deploy from Template → "n8n" 검색
3. 환경변수 자동 셋업 됨. `WEBHOOK_URL`만 본인 URL로 확인
4. 첫 1개월 무료 크레딧 $5, 이후 약 $5~10/월

### Option B: Hetzner + Docker (비용 최소, 30분)

```bash
# 서버: Hetzner CX11 (€4/월)
docker run -d --restart=always \
  --name n8n \
  -p 5678:5678 \
  -e GENERIC_TIMEZONE="Asia/Seoul" \
  -e WEBHOOK_URL="https://n8n.yourdomain.com" \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

Cloudflare Tunnel 또는 Caddy로 HTTPS 종단.

### Option C: n8n Cloud

https://n8n.io/cloud — 월 $20부터. 학습 안 하고 바로 사용.

---

### 첫 로그인

n8n URL 접속 → owner 계정 생성 → 워크스페이스 진입.

---

## 2. Notion 워크스페이스 + DB 6종

### 2.1 Integration 만들기

1. https://www.notion.so/my-integrations → **+ New integration**
2. Name: `marketing-lab`
3. Associated workspace: 작업할 워크스페이스
4. Type: **Internal**
5. Capabilities: **Read / Update / Insert content**
6. **Internal Integration Secret** 복사 → 어딘가 안전한 곳에 보관 (n8n credentials에 곧 등록)

### 2.2 DB 6종 생성

Notion에서 새 페이지 **Full page database** 6개 만든다. 스키마는 [`../notion-schemas/`](../notion-schemas/) 참고:

| DB | 용도 | 사용 자동화 |
|---|---|---|
| [`ContentIdeas`](../notion-schemas/content-ideas.md) | 글감·페인포인트 | 01, 02, 05 |
| [`ContentDrafts`](../notion-schemas/content-drafts.md) | 콘텐츠 초안·발행 상태 | 02, 05, 08 |
| [`Leads`](../notion-schemas/leads.md) | 리드 + enrich 데이터 | 07 |
| [`CampaignMetrics`](../notion-schemas/campaign-metrics.md) | 광고·이메일·LP 성과 | 06, 09, 10 |
| [`Inbox`](../notion-schemas/inbox.md) | 댓글·DM·문의 통합 | 03, 09 |
| [`CompetitorTimeline`](../notion-schemas/competitor-timeline.md) | 경쟁사 변화 시계열 | 04, 06, 09 |

각 DB마다:
1. 스키마 문서 보고 속성 추가
2. 페이지 우상단 `...` → **Add connections** → `marketing-lab` integration 추가
3. URL에서 32자 hex DB ID 추출 → `.env`에 저장

---

## 3. Slack 워크스페이스 + 채널

### 3.1 채널 5개 생성

| 채널 | 용도 |
|---|---|
| `#automation-log` | 모든 자동화의 실행 로그·에러 |
| `#content-ideas` | 01 일간 다이제스트 |
| `#inbox` | 03 댓글·DM·문의 통합 |
| `#competitor-intel` | 04 경쟁사 변화 알림 |
| `#weekly-report` | 09 주간 리포트 |

### 3.2 봇 앱 생성

1. https://api.slack.com/apps → **Create New App** → From scratch
2. App Name: `marketing-lab-bot`, workspace 선택
3. 좌측 **OAuth & Permissions** → Bot Token Scopes 추가:
   - `chat:write`
   - `chat:write.public`
   - `channels:read` (채널 ID 조회용)
4. 상단 **Install to Workspace** → 승인
5. **Bot User OAuth Token** (`xoxb-...`) 복사 → `.env`로
6. 각 채널에 봇 초대: 채널 들어가서 `/invite @marketing-lab-bot`

---

## 4. API 키 발급

### Anthropic (필수)

1. https://console.anthropic.com → API Keys → **Create Key**
2. 결제 정보 등록 (월 $5 미만으로 시작 가능)
3. 키 (`sk-ant-...`) 복사 → `.env`

### OpenAI (선택)

Whisper(02 유튜브 팩토리에서 자막 없는 영상용)와 백업 LLM용. 지금은 건너뛰고 02 진입 시 추가해도 됨.

---

## 5. n8n credentials 등록

n8n UI 좌측 메뉴 → **Credentials** → **+ Add Credential**

| Credential 타입 | 입력값 | 자동화에서 참조명 |
|---|---|---|
| **Notion API** | Internal Integration Secret | `Notion` |
| **Slack API** | Bot User OAuth Token (`xoxb-...`) | `Slack` |
| **Anthropic API** | `sk-ant-...` | `Anthropic` |

각각 저장 후 **Test** 버튼으로 연결 확인.

---

## 6. 환경변수 (.env)

레포 루트의 `.env.example` 복사:

```bash
cp .env.example .env
```

채워 넣을 값:
```bash
NOTION_DB_CONTENT_IDEAS=<32자 hex>
NOTION_DB_CONTENT_DRAFTS=<32자 hex>
NOTION_DB_LEADS=<32자 hex>
NOTION_DB_CAMPAIGN_METRICS=<32자 hex>
NOTION_DB_INBOX=<32자 hex>
NOTION_DB_COMPETITOR_TIMELINE=<32자 hex>

SLACK_CHANNEL_LOG=#automation-log
SLACK_CHANNEL_IDEAS=#content-ideas
SLACK_CHANNEL_INBOX=#inbox
SLACK_CHANNEL_COMPETITOR=#competitor-intel
SLACK_CHANNEL_REPORT=#weekly-report
```

### n8n에 환경변수 주입

**Railway**: Project → Variables 탭에 각 KEY=VALUE 추가
**Docker**: 컨테이너 실행 시 `-e KEY=VALUE` 또는 `--env-file .env`
**n8n Cloud**: Settings → Environment variables (Pro 플랜 이상)

n8n 재시작 후 워크플로에서 `{{ $env.NOTION_DB_CONTENT_IDEAS }}` 형태로 참조 가능.

---

## 7. Hello World 워크플로

검증용 최소 워크플로. [`workflows/hello-world.json`](./workflows/hello-world.json)을 임포트.

### 임포트

n8n → 우상단 `...` → **Import from File** → `hello-world.json`

### 노드 구성

```
[Cron: 매분]
   ▼
[Slack: 메시지 발송]
```

### 실행

1. 캔버스 우상단 **Execute Workflow** 클릭
2. `#automation-log` 채널에 "✅ Layer 0 OK — n8n is alive" 메시지 도착 확인

도착하지 않으면:
- Slack credential 다시 Test
- 봇이 채널에 초대되어 있는지 확인 (`/invite @marketing-lab-bot`)
- 채널 이름 오타 확인 (`#` 포함)

---

## 8. 활성화

Hello World가 수동 실행으로 도착 확인되면:

1. 워크플로 우상단 토글을 **Active**로
2. 매분 메시지가 오면 정상 → **확인 후 즉시 비활성화** (스팸됨)
3. 또는 Cron을 `0 9 * * *` (매일 9시)로 변경

---

## 9. Layer 0 완료 시그널

✅ Notion에 DB 6개가 보임 (ContentIdeas, ContentDrafts, Leads, CampaignMetrics, Inbox, CompetitorTimeline)
✅ Slack에 채널 5개와 봇이 있음
✅ n8n에 credential 3개가 등록됨
✅ Hello World가 Slack에 메시지를 보냄
✅ `.env`가 채워져 있고 `.gitignore`에 포함됨

위 5개 다 ✅면 어느 자동화든 진행 가능. 권장 순서는 [`../roadmap.md`](../roadmap.md) 참고.

---

## 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| n8n이 환경변수를 못 읽음 | n8n 재시작 안 함 | 컨테이너/서비스 재시작 |
| Notion `unauthorized` | DB에 integration 연결 안 됨 | DB 페이지 `...` → Add connections |
| Slack `channel_not_found` | 봇이 채널 미가입 | `/invite @marketing-lab-bot` |
| Anthropic `invalid_api_key` | 키 복사 시 공백 포함 | 다시 발급, trim 확인 |
| n8n 워크플로 import 실패 | n8n 버전 미스매치 | n8n 1.50.0 이상 권장 |
