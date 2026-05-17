# 03 통합 인박스 — 세팅 가이드

`workflows/unified-inbox.json`을 임포트해서 IG 댓글·Gmail 문의를 5분마다 모으고 분류·답변 초안까지.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) 완료 + Inbox DB 생성

**소요 시간**: 2~3시간 (IG Business 셋업이 가장 오래 걸림)

---

## 0. 준비물 체크

- [ ] Layer 0 완료 — 특히 [`Inbox` DB](../../docs/notion-schemas/inbox.md) 생성
- [ ] **Instagram Business 계정** + 연결된 Facebook 페이지
- [ ] Meta 개발자 계정 + 앱 생성 권한
- [ ] **Google Cloud 프로젝트** + Gmail API 활성화
- [ ] n8n에 Gmail OAuth credential 등록 가능 상태

---

## 1. Instagram Business + Meta App 셋업

가장 복잡한 단계. 차근차근.

### 1.1 IG 계정을 Business 계정으로 전환

IG 앱 → 프로필 → 메뉴 → Settings → Account type → **Switch to Business Account**.

이후 Facebook 페이지에 연결되어야 함 (Facebook Business에서 페이지 생성 필요).

### 1.2 Meta 개발자 앱 생성

1. https://developers.facebook.com → My Apps → **Create App**
2. App type: **Business**
3. App name: `marketing-lab-inbox`
4. 생성 후 좌측 **Add products** → **Instagram Graph API**, **Facebook Login** 추가

### 1.3 Permissions 신청

App Review → Permissions에서 다음 요청:
- `instagram_basic`
- `instagram_manage_comments`
- `pages_show_list`
- `pages_read_engagement`
- `business_management`

⚠️ MVP 검증 단계에서는 본인 계정만 사용 가능 (App Review 통과 전). 본인 IG 계정을 **Roles → Testers**에 추가하면 권한 없이도 작동.

### 1.4 Access Token 발급 (Long-lived)

1. Graph API Explorer (https://developers.facebook.com/tools/explorer)
2. Application 선택: `marketing-lab-inbox`
3. User access token 생성 → 위 권한 모두 체크
4. 단기 토큰 발급 (1시간 유효)
5. 다음 URL로 장기 토큰 교환 (60일):
```
https://graph.facebook.com/v19.0/oauth/access_token?
  grant_type=fb_exchange_token&
  client_id={APP_ID}&
  client_secret={APP_SECRET}&
  fb_exchange_token={SHORT_TOKEN}
```
6. 응답의 `access_token`을 `.env`에:
```bash
META_GRAPH_ACCESS_TOKEN=EAAxxxxx...
```

### 1.5 Instagram User ID 조회

```bash
curl "https://graph.facebook.com/v19.0/me/accounts?access_token={TOKEN}"
```

응답에서 본인 Facebook 페이지의 `id` (Page ID) 확인. 그 다음:

```bash
curl "https://graph.facebook.com/v19.0/{PAGE_ID}?fields=instagram_business_account&access_token={TOKEN}"
```

응답의 `instagram_business_account.id`가 IG User ID. `.env`에:
```bash
META_PAGE_ID={Page ID}
META_IG_USER_ID={IG User ID}
```

### 1.6 토큰 만료 대비

장기 토큰은 60일 후 만료. 두 가지 옵션:
- **수동 갱신**: 60일마다 1.4 단계 반복
- **System User 토큰**: Business Manager → System Users로 영구 토큰 발급 (권장)

---

## 2. Gmail OAuth 셋업

### 2.1 Google Cloud 프로젝트

1. https://console.cloud.google.com → 프로젝트 생성: `marketing-lab`
2. APIs & Services → Library → **Gmail API** 검색 → Enable

### 2.2 OAuth Consent Screen

1. APIs & Services → OAuth consent screen
2. User Type: **External** (개인 Gmail용)
3. App name: `marketing-lab`, support email 입력
4. Scopes 추가:
   - `https://www.googleapis.com/auth/gmail.readonly`
   - `https://www.googleapis.com/auth/gmail.modify` (라벨 변경용, v2)
5. Test users에 본인 Gmail 추가

### 2.3 OAuth Client ID

1. APIs & Services → Credentials → **Create Credentials → OAuth client ID**
2. Application type: **Web application**
3. Authorized redirect URIs에 n8n의 OAuth callback URL 추가:
   - n8n Cloud: `https://oauth.n8n.cloud/oauth2/callback`
   - 셀프호스팅: `https://your-n8n-domain/rest/oauth2-credential/callback`
4. Client ID + Client Secret 복사

### 2.4 n8n에 Gmail Credential 등록

1. n8n → Credentials → **+ Add Credential** → **Gmail OAuth2 API**
2. Client ID / Client Secret 입력
3. **Sign in with Google** 클릭 → Google 인증 → 권한 승인
4. Test 통과 확인

### 2.5 라벨 만들기 (선택)

`#inbox-monitor`처럼 모니터링할 라벨을 Gmail에서 만들면, 필터로 특정 메일만 자동 라벨링 가능. `.env`:
```bash
GMAIL_LABEL=inbox-monitor
```

워크플로의 `Gmail: Unread` 노드 query를 `label:inbox-monitor is:unread`로 변경.

MVP는 모든 unread 메일 대상.

---

## 3. Notion `Inbox` DB 확인

[스키마](../../docs/notion-schemas/inbox.md) 모든 속성이 정확한 이름·타입으로 있어야 함. 특히:
- `external_id` (Rich text) — 중복 방지 핵심
- `channel` (Select) — 옵션에 `instagram_comment`, `gmail` 포함

Integration 연결: DB 페이지 `...` → Add connections → `marketing-lab`.

`.env`:
```bash
NOTION_DB_INBOX=<32자 hex>
```

---

## 4. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/unified-inbox.json`

### 임포트 후 확인

- [ ] **IG: Recent Media / IG: Comments per Media**: credentials는 없고 URL에 `$env.META_GRAPH_ACCESS_TOKEN` 참조 — 환경변수 채워졌는지 확인
- [ ] **Gmail: Unread**: Gmail OAuth credential 선택
- [ ] **Already In Notion? / Notion: Inbox Append**: Notion credential 선택
- [ ] **Claude: Classify + Draft**: Anthropic credential 선택
- [ ] **Slack: #inbox**: Slack credential 선택, 채널 `$env.SLACK_CHANNEL_INBOX`
- [ ] **Config** 노드의 `brand_voice`를 본인 브랜드에 맞게 수정

---

## 5. 첫 실행 (수동 테스트)

### 단계별 테스트

1. **IG branch만 먼저**:
   - `IG: Recent Media` 노드 우클릭 → Execute Node
   - 응답에 최근 미디어 10개 보이는지

2. **Gmail branch**:
   - 본인 Gmail에 unread 메일이 있어야 함 (안 보내봤으면 테스트 메일 1통 전송)
   - `Gmail: Unread` 실행 → 결과 확인

3. **전체 실행**: `Every 5 min` 트리거 노드 → Execute Workflow
   - Notion `Inbox` DB에 신규 행 생성 확인
   - Slack `#inbox` 채널에 메시지 도착 확인

### 통과 시그널 (각 노드)

| 노드 | 확인 |
|---|---|
| IG: Recent Media | `data` 배열에 미디어 객체 |
| Flatten IG Comments | 최근 15분 내 댓글만 (lookback 필터 작동) |
| Gmail: Unread | unread 메시지 객체 |
| Merge Sources | IG + Gmail 합쳐진 배열 |
| Already In Notion? | 신규 메시지는 빈 결과 |
| Claude: Classify + Draft | JSON 응답 |
| Notion: Inbox Append | DB에 행 생성 (`status=new`) |
| Slack: #inbox | 채널에 메시지 도착 |

---

## 6. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| Meta API `(#190) Invalid OAuth token` | 토큰 만료 (60일) | 1.4 단계 재실행, System User 토큰으로 영구화 권장 |
| Meta API `Permissions error` | App Review 미통과 + tester 미등록 | App → Roles → Testers에 본인 IG 계정 추가 |
| `Flatten IG Comments` 0건 | lookback_minutes 짧음 또는 최근 댓글 없음 | Config의 `look_back_minutes`를 1440(1일)으로 늘려 테스트 |
| Gmail unauthorized_client | OAuth consent screen에 본인 이메일이 test user로 없음 | Google Cloud Console에서 추가 |
| Notion `external_id` 중복 못 잡음 | 속성 타입이 `rich_text`가 아닐 때 | DB 스키마 재확인 |
| Slack 메시지 너무 김 | 댓글이 매우 긴 경우 | Build Slack Message Code 노드의 `slice(0, 500)` 조정 |

---

## 7. 활성화

수동 테스트 통과 후 워크플로 토글 **Active**. 5분 주기로 자동 폴링.

⚠️ Meta API 호출 빈도 주의 — 본 워크플로는 5분 × IG 미디어 10개 = 시간당 120회 호출. App 한도(시간당 200회 per user)에 가깝지만 안전.

---

## 8. 운영 팁

### Slack에서 답변 발송 흐름 (MVP)

1. `#inbox`에 신규 메시지 알림 도착
2. 답변 초안 검토 → 마음에 들면 그대로 복사
3. 각 채널에서 직접 답변 (IG 앱, Gmail 등)
4. Notion `Inbox` DB에서 해당 행 `status=replied`로 변경

### 발송 자동화 (v2)

PRD v2 항목 중:
1. Slack block kit으로 ✅ 발송 / ❌ 무시 / 🙋 사람 버튼
2. ✅ 누르면 채널 API로 자동 발송 (IG comments reply / Gmail send)
3. confidence > 0.9면 사람 개입 없이 자동 발송 옵션

### 비용 추정

| 항목 | 일 100건 기준 |
|---|---|
| Claude API | ≈ $0.50/일 = 월 $15 |
| Meta Graph API | 무료 (한도 내) |
| Gmail API | 무료 (한도 내) |
| **합계** | ≈ **$15/월** |

### 채널 추가 (v2)

- YouTube 댓글: YouTube Data API + 별도 node
- 블로그 댓글: WordPress webhook → 별도 trigger 워크플로 또는 추가 source
- X 멘션·DM: X API + 별도 node

확장 시 `Merge Sources` 노드 input 개수만 늘리면 나머지(Claude·Notion·Slack)는 그대로 재사용 가능 — 모놀리식 설계가 아닌 source-agnostic 처리.

---

## 9. 다음 단계

- 첫 1주 답변 초안 채택률(`draft_reply` 그대로 발송한 비율) 추적
- 채택률 < 50% → `prompts/inbox-classify-reply.md` 브랜드 보이스 보강, few-shot 예시 추가
- 채택률 > 70% → v2 Slack block kit 자동 발송 단계
- 다른 채널 추가는 그 후

또는 다음 자동화([04 경쟁사 인텔](../04-competitor-intel/) — 같은 Week 4)로 이동.
