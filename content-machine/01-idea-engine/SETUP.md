# 01 아이디어 엔진 — 세팅 가이드

`workflows/idea-engine.json`을 임포트해서 돌리기까지 필요한 모든 것.

소요 시간 추정: **2~3시간** (API 키 발급 포함, 처음이라면)

---

## 0. 준비물 체크

- [ ] n8n 인스턴스 (셀프호스팅 또는 cloud)
- [ ] Notion 워크스페이스 (Integration 생성 권한)
- [ ] Anthropic API 키 ([console.anthropic.com](https://console.anthropic.com))
- [ ] X(Twitter) 개발자 계정 — Basic tier 필요 ($200/월) **또는** 이번 MVP에서는 X 노드 비활성화하고 Reddit만 사용
- [ ] Slack 워크스페이스 + 봇 토큰

---

## 1. Notion DB 생성

Notion에서 **새 DB(Full page)** 생성, 이름 `ContentIdeas`.

### 필수 속성

| 속성 이름 | 타입 | 비고 |
|---|---|---|
| `Name` | Title | 자동 생성된 제목 (Notion 기본) |
| `pain_point` | Rich text | LLM 추출 페인포인트 |
| `source` | Select | `reddit`, `x`, `youtube`, `naver` 옵션 미리 추가 |
| `source_url` | URL | 원본 링크 |
| `keyword` | Select | 트래킹 키워드 (옵션은 동적 추가) |
| `score` | Number | LLM이 매긴 0-10 점수 |
| `engagement` | Number | 원본 인게이지먼트 (정렬용) |
| `status` | Select | `new`, `triaged`, `queued`, `drafted`, `published`, `archived` |
| `created_at` | Created time | Notion 기본 자동 |

### Notion Integration 연결

1. https://www.notion.so/my-integrations → New integration
2. Name: `marketing-lab`, type: Internal, capabilities: read/write content
3. Internal Integration Secret 복사 → n8n credentials에 저장 (`Notion API`)
4. `ContentIdeas` DB 페이지 우상단 `...` → **Add connections** → `marketing-lab` 추가
5. DB URL에서 ID 추출: `notion.so/<workspace>/<DATABASE_ID>?v=...` — `<DATABASE_ID>` 부분

n8n 환경변수에 등록:
```
NOTION_DB_CONTENT_IDEAS=<32자 hex>
```

---

## 2. Reddit API 셋업

공개 검색은 OAuth 없이도 가능하지만 **rate limit이 빡빡함** (분당 ~10회). 운영 시는 OAuth 권장.

### Option A: OAuth (권장)

1. https://www.reddit.com/prefs/apps → Create app → **script** 타입
2. redirect URI: `http://localhost:8080` (script에선 안 씀)
3. client ID/secret 복사
4. n8n credentials → `Reddit OAuth2 API` 등록
5. 워크플로의 `Reddit Search` 노드 Authentication을 `OAuth2`로 변경

### Option B: 익명 (MVP에서는 OK)

워크플로 그대로 사용. 단, User-Agent 헤더는 의미 있는 값으로 유지 (현재 `marketing-lab-idea-engine/0.1`).

---

## 3. X(Twitter) API 셋업

### Option A: 공식 API ($200/월 Basic)

1. https://developer.x.com → 프로젝트 + 앱 생성
2. OAuth 2.0 Bearer Token 발급
3. n8n credentials → 새 `Twitter OAuth2 API` (Bearer만으로 충분)
4. 워크플로 import 시 자동으로 credential 연결 요청

### Option B: 우회 (MVP 비용 절감)

X 노드를 비활성화하고 Reddit만으로 시작. 추후:
- Apify `Twitter Scraper` actor ($) — n8n에서 HTTP Request 노드로 호출
- 또는 v2에서 X 추가

비활성화 방법: `X Search` 노드 우클릭 → **Deactivate Node**. 워크플로는 Reddit만으로도 작동 (Merge가 Reddit input만 받음).

---

## 4. Anthropic 노드 셋업

1. https://console.anthropic.com → API Keys → Create
2. n8n credentials → `Anthropic API` → 키 입력
3. 모델 선택은 워크플로의 `Claude: Extract Pain Point` 노드에서:
   - 기본: `claude-sonnet-4-6`
   - 비용 절감: `claude-haiku-4-5-20251001` (품질 시험 필요)
   - 고품질: `claude-opus-4-7` (비용 ~5배)

---

## 5. Slack 봇 셋업

1. https://api.slack.com/apps → Create New App → From scratch
2. Bot Token Scopes 추가:
   - `chat:write`
   - `chat:write.public`
3. Install to workspace → Bot User OAuth Token (`xoxb-...`) 복사
4. n8n credentials → `Slack API` → 토큰 입력
5. Slack에서 `#content-ideas` 채널 생성 → 채널 설정에서 봇 초대

---

## 6. 워크플로 임포트

n8n UI → 우상단 `...` → **Import from File** → `workflows/idea-engine.json` 선택.

임포트 후 확인 사항:
- [ ] 각 노드의 credential 드롭다운에서 위에서 만든 credential 선택
- [ ] `Config` 노드의 키워드·서브레딧 값을 본인 도메인에 맞게 수정
- [ ] `Notion: Append Idea` 노드의 DB ID가 `$env.NOTION_DB_CONTENT_IDEAS` 환경변수를 참조 → n8n Settings → Environment variables 설정 확인
- [ ] `Slack` 노드의 채널 이름 확인

---

## 7. 첫 실행 (수동 테스트)

1. 워크플로 캔버스 우상단 **Execute Workflow** 클릭
2. 각 노드 통과 확인:
   - Reddit Search: `data.children` 배열에 결과 있어야 함
   - Normalize + Dedup: 30건 이하로 정규화된 객체
   - Claude: 각 건마다 JSON 응답
   - Score Filter: score ≥ 6만 통과
   - Notion: 신규 페이지 생성 확인 (Notion DB 확인)
   - Slack: 메시지 도착 확인

### 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| Reddit 429 | rate limit | OAuth로 전환, 또는 호출 빈도 축소 |
| Claude `Invalid JSON` | LLM이 가끔 markdown 감쌈 | `Parse LLM Output` Code 노드의 JSON.parse 실패 시 fence 제거 로직 추가 (현재는 fail-safe로 empty array 반환) |
| Notion `property does not exist` | DB 속성 이름 불일치 | DB 속성 이름이 SETUP.md 표와 정확히 일치해야 함 (대소문자·언더스코어 포함) |
| Slack `not_in_channel` | 봇이 채널에 안 들어감 | 채널에 `/invite @marketing-lab` |

---

## 8. 활성화 (자동 실행)

수동 실행 정상 작동 확인 후:
1. 워크플로 우상단 토글을 **Active**로 전환
2. 매일 07:00에 자동 실행
3. 첫 1주는 매일 결과를 사람이 보고 점수/프롬프트 튜닝 (`prompts/extract-painpoint.md` 마지막 섹션 참고)

---

## 9. 키워드·임계값 수정 위치

`Config` 노드에서만 수정:
- `keywords`: 모니터링할 키워드 (5개 권장, 너무 많으면 LLM 비용 ↑)
- `subreddits`: 검색 대상 서브레딧
- `min_score`: 다이제스트/Notion 적재 임계값 (시작은 6, 노이즈 많으면 7로)

다른 노드는 모두 이 값을 expression으로 참조하므로 한 곳에서만 바꾸면 된다.

---

## 10. 다음 단계

이 워크플로가 1주일 안정적으로 굴면:

1. **PRD `MVP 범위 (제외)` 항목 중 하나 추가**: 네이버 카페 또는 유튜브 댓글 소스
2. **02 유튜브 팩토리**로 진행 (선정된 글감으로 영상 기획 → 콘텐츠 자산화)
3. v2 다이제스트(`prompts/daily-digest.md` 참고) — 테마 클러스터링 강화
