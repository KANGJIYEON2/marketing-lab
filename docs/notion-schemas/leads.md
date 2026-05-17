# Leads DB

07 리드 시퀀스가 폼 제출을 받아 enrich + 분류 + 시퀀스 상태를 추적.

**사용 자동화**: 07(쓰기), 09(읽기 — KPI 집계)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | 리드 이름 (폼 입력) |
| `email` | Email | ✓ | 이메일 |
| `company` | Rich text | | enrich 결과 |
| `role` | Rich text | | 직책 (enrich) |
| `domain` | URL | | 회사 도메인 |
| `linkedin_url` | URL | | LinkedIn 프로필 |
| `source` | Select | ✓ | `landing_page`, `tally`, `typeform`, `import`, `referral` |
| `utm_source` | Rich text | | UTM 소스 |
| `utm_campaign` | Rich text | | UTM 캠페인 |
| `persona` | Select | ✓ | `founder`, `marketer`, `dev`, `other` |
| `score` | Number | ✓ | 0-100 |
| `hot_signal` | Checkbox | | 즉시 영업 핫리드 |
| `sequence_status` | Select | ✓ | `enrolled`, `day0_sent`, `day2_sent`, `day5_sent`, `replied`, `unsubscribed`, `converted` |
| `last_action_at` | Date | | 마지막 메일 발송/답장 시간 |
| `notes` | Rich text | | 메모 |
| `created_at` | Created time | (자동) | |

## Select 옵션

### `source`
- `landing_page` (gray)
- `tally` (purple)
- `typeform` (blue)
- `import` (yellow)
- `referral` (green)

### `persona`
- `founder` (orange)
- `marketer` (pink)
- `dev` (purple)
- `other` (gray)

### `sequence_status`
- `enrolled` (gray) — 폼 제출됨
- `day0_sent` (blue)
- `day2_sent` (blue)
- `day5_sent` (blue)
- `replied` (green) — 답장 옴
- `unsubscribed` (red)
- `converted` (purple) — 미팅 잡힘/구매 등

## 뷰 권장

- **Hot leads**: filter `hot_signal = true AND sequence_status != converted`, sort `score desc`
- **In sequence**: filter `sequence_status in [day0_sent, day2_sent, day5_sent]`
- **Replied (action needed)**: filter `sequence_status = replied`
- **By persona**: group `persona`

## 환경변수

`NOTION_DB_LEADS`

## 개인정보 주의

이 DB는 **개인정보가 들어있다**. 다음 보호:
- 워크스페이스 권한: 작업자 외 공유 금지
- Notion Integration 권한도 `marketing-lab` 외 금지
- 백업 export 시 암호화 + 별도 저장소
