# Inbox DB

03 통합 인박스가 IG 댓글·Gmail 문의 등을 채널·작성자별로 적재. 답변 상태 추적 + 향후 분석용.

**사용 자동화**: 03(쓰기·업데이트), 09(읽기 — 응답 시간/누락률 KPI)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | `{channel}: {author} — {content 앞 40자}` |
| `channel` | Select | ✓ | `instagram_comment`, `instagram_dm`, `gmail`, `youtube_comment`, `blog_comment`, `x_mention` |
| `author` | Rich text | ✓ | 작성자 표시명 또는 ID |
| `content` | Rich text | ✓ | 원본 메시지 본문 |
| `source_url` | URL | | 원본 댓글/메시지 링크 |
| `external_id` | Rich text | ✓ | 채널 시스템의 고유 ID (중복 방지) |
| `received_at` | Date | ✓ | 채널에 도착한 시간 |
| `intent` | Select | ✓ | `question`, `complaint`, `praise`, `spam`, `sales_lead`, `other` |
| `urgency` | Select | ✓ | `high`, `medium`, `low` |
| `sentiment` | Select | ✓ | `positive`, `neutral`, `negative` |
| `draft_reply` | Rich text | | LLM이 생성한 답변 초안 |
| `confidence` | Number | | 분류·초안의 신뢰도 0-1 |
| `status` | Select | ✓ | `new`, `replied`, `ignored`, `escalated` |
| `replied_by` | Person | | 답변자 (수동 발송 시) |
| `replied_at` | Date | | 답변 시간 |

## Select 옵션

### `channel`
- `instagram_comment` (pink)
- `instagram_dm` (purple)
- `gmail` (red)
- `youtube_comment` (red)
- `blog_comment` (gray)
- `x_mention` (black)

### `intent`
- `question` (blue) — 정보 요청, 도움 요청
- `complaint` (red) — 불만·문제 보고
- `praise` (green) — 칭찬·후기
- `spam` (gray) — 광고·봇
- `sales_lead` (orange) — 구매·협업 문의
- `other` (gray)

### `urgency`
- `high` (red) — 30분 내 응답 권장 (불만, 영업 기회)
- `medium` (yellow) — 4시간 내
- `low` (gray) — 24시간 내

### `sentiment`
- `positive` (green)
- `neutral` (gray)
- `negative` (red)

### `status`
- `new` (blue) — 신규 수집
- `replied` (green) — 답변 완료
- `ignored` (gray) — 무시 (스팸 등)
- `escalated` (red) — 사람이 직접 처리해야 함

## 뷰 권장

- **Triage queue**: filter `status = new`, sort `urgency desc, received_at asc`
- **Hot (high urgency unanswered)**: filter `status = new AND urgency = high`
- **Today**: filter `received_at >= today`
- **By channel**: group `channel`
- **Response time analytics**: `replied_at - received_at` 계산용 (09 리포트)

## 데이터 수명 정책

- `status = ignored` 행은 30일 후 archive
- 그 외는 영구 보관 (학습용)
- 개인정보(`author`, `content`) 포함 가능 → 워크스페이스 권한 제한

## 환경변수

`NOTION_DB_INBOX`

## 본문 (페이지 콘텐츠)

워크플로가 페이지 본문에 자동 추가:
- `H2 Original Content` + 전체 본문
- `H2 Draft Reply` + LLM 답변 초안 (편집 가능)
- `H2 Classification Reasoning` (선택)
