# CampaignMetrics DB

광고·이메일·LP의 일간/주간 성과를 한 곳에 적재. 09 주간 리포트가 읽고, 06·10이 비교 기준으로 사용.

**사용 자동화**: 06(쓰기 — 광고 성과), 07(쓰기 — 이메일 발송 성과), 09(읽기 — 집계), 10(쓰기 — A/B 결과)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | 캠페인/실험명 (예: `Meta-Spring-2026-Headline-A`) |
| `channel` | Select | ✓ | `meta_ads`, `google_ads`, `email`, `lp_experiment`, `organic_social` |
| `entity_type` | Select | ✓ | `campaign`, `ad_set`, `ad`, `email_send`, `lp_variant` |
| `entity_id` | Rich text | | 외부 시스템 ID (Meta ad_id 등) |
| `date` | Date | ✓ | 데이터 기준일 |
| `period` | Select | ✓ | `daily`, `weekly` |
| `spend` | Number | | 비용 (USD) |
| `impressions` | Number | | |
| `clicks` | Number | | |
| `conversions` | Number | | |
| `revenue` | Number | | 매출 (있는 경우) |
| `ctr` | Number | | clicks/impressions |
| `cpa` | Number | | spend/conversions |
| `roas` | Number | | revenue/spend |
| `notes` | Rich text | | 변경 사항 메모 (예: "Headline 변형 A로 교체") |
| `experiment_id` | Rich text | | 10의 실험 추적용 |
| `winner` | Checkbox | | A/B 위너 표시 |
| `created_at` | Created time | (자동) | |

## Select 옵션

### `channel`
- `meta_ads` (blue)
- `google_ads` (red)
- `email` (yellow)
- `lp_experiment` (purple)
- `organic_social` (pink)

### `entity_type`
- `campaign` (orange)
- `ad_set` (yellow)
- `ad` (gray)
- `email_send` (blue)
- `lp_variant` (purple)

### `period`
- `daily` (gray)
- `weekly` (blue)

## 뷰 권장

- **This week's ads**: filter `channel in [meta_ads, google_ads] AND period = daily AND date >= last_7_days`
- **Active experiments**: filter `entity_type = lp_variant AND winner = false`
- **Winners**: filter `winner = true`, sort `date desc`
- **Weekly trend**: filter `period = weekly`, sort `date desc`

## 데이터 수명 정책

- `daily` 행은 90일 보관 후 archive (별도 DB로 이동 또는 hidden)
- `weekly` 행은 영구 보관
- 09 주간 리포트는 `weekly` 행만 참조

## 환경변수

`NOTION_DB_CAMPAIGN_METRICS`
