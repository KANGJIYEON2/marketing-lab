# Notion DB 스키마

마케팅 자동화 10종이 공유하는 DB 4종 스키마 명세. Layer 0 셋업 시 이 문서대로 Notion DB를 만든다.

| DB | 주 사용 자동화 | 환경변수 |
|---|---|---|
| [`ContentIdeas`](./content-ideas.md) | 01, 02, 05 | `NOTION_DB_CONTENT_IDEAS` |
| [`ContentDrafts`](./content-drafts.md) | 02, 05, 06, 08 | `NOTION_DB_CONTENT_DRAFTS` |
| [`Leads`](./leads.md) | 07, 09 | `NOTION_DB_LEADS` |
| [`CampaignMetrics`](./campaign-metrics.md) | 06, 07, 09, 10 | `NOTION_DB_CAMPAIGN_METRICS` |
| [`Inbox`](./inbox.md) | 03, 09 | `NOTION_DB_INBOX` |
| [`CompetitorTimeline`](./competitor-timeline.md) | 04, 06, 09 | `NOTION_DB_COMPETITOR_TIMELINE` |

## 공통 규칙

1. **DB 이름은 영문 정확히 일치** (`ContentIdeas`, `ContentDrafts`, `Leads`, `CampaignMetrics`). 한글 추가 금지 — 자동화 코드가 깨질 수 있음.
2. **속성 이름도 영문 + snake_case**. 스키마 표의 이름과 글자 단위로 동일해야 함.
3. **Select 옵션**은 사전 정의된 것만 사용. 자동화가 새 옵션을 만들면 노이즈로 누적됨.
4. **Integration 연결 필수**: 각 DB 페이지 우상단 `...` → Add connections → `marketing-lab` 추가.

## 스키마 변경 시

- 자동화 코드가 깨질 수 있으니 신중히
- 변경 전 영향 자동화 PRD 확인
- 변경 후 본 문서 + 영향 PRD 모두 업데이트
- 가능하면 **속성 추가만**, 이름 변경/삭제는 마이그레이션 동반
