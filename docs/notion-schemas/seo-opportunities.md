# SEOOpportunities DB

05 SEO 헌터가 발견한 11~20위 기회 키워드와 그에 대한 업데이트 액션 아이템.

**사용 자동화**: 05(쓰기), 09(읽기 — 진행률 KPI)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | `{keyword} — {url path}` |
| `url` | URL | ✓ | 내 사이트의 글 URL |
| `keyword` | Rich text | ✓ | 11-20위 키워드 |
| `position` | Number | ✓ | GSC 평균 순위 (소수점) |
| `impressions` | Number | ✓ | 28일 노출수 |
| `clicks` | Number | ✓ | 28일 클릭수 |
| `ctr` | Number | | clicks / impressions |
| `priority_score` | Number | ✓ | `impressions × (top10_avg_ctr - current_ctr)` |
| `action_items` | Rich text | ✓ | Claude 추출 액션 (\n으로 구분된 bullet) |
| `missing_sections` | Rich text | | Top 10 글에는 있는데 내 글에 없는 섹션 |
| `weak_points` | Rich text | | 내 글의 약점 |
| `status` | Select | ✓ | `new`, `triaged`, `in_progress`, `done`, `archived` |
| `assigned_to` | Person | | 작업 담당 |
| `done_at` | Date | | 작업 완료 시간 |
| `result_position` | Number | | 작업 후 측정된 새 순위 (4주 후 09가 업데이트) |
| `captured_at` | Created time | (자동) | |

## Select 옵션

### `status`
- `new` (blue) — 자동 추가됨
- `triaged` (yellow) — 사람이 검토함
- `in_progress` (orange) — 글 업데이트 진행 중
- `done` (green) — 발행됨, 4주 측정 대기
- `archived` (gray) — 폐기 (우선순위 낮음 또는 작업 안 함)

## 뷰 권장

- **Triage queue**: filter `status = new`, sort `priority_score desc`
- **This week's targets**: filter `status in [triaged, in_progress]`, sort `priority_score desc`
- **Recently shipped**: filter `status = done`, sort `done_at desc`
- **Win rate**: filter `status = done AND result_position is not null`, calculate `position - result_position` (포지션 변화)

## priority_score 산식

```
priority_score = impressions × (avg_top10_ctr - current_ctr)
```

- `avg_top10_ctr`: 일반적으로 1페이지 평균 CTR (10위는 ~3%, 1위는 ~30%). 단순화해서 0.12 사용.
- `current_ctr`: 이 키워드의 현재 CTR
- 결과: "1페이지 진입 시 추가로 얻을 클릭 수"의 추정치

## 본문 (페이지 콘텐츠)

워크플로가 페이지 본문에 자동 추가:
- `H2 Top 10 SERP Summary` — 1페이지 상위 글들의 핵심 토픽 요약
- `H2 Action Items` — 액션 아이템 bullet
- `H2 Missing Sections` — 내 글에 추가할 섹션 제안
- `H2 Weak Points` — 내 글의 약점

## 데이터 수명 정책

- 같은 (url, keyword) 조합 중복 추가 방지 — `external_id`처럼 사용
- 매주 실행 시 새 기회만 추가, 기존 `new` 행이 있으면 skip (사람이 처리할 시간 줌)
- 4주 이상 `new` 상태 → `archived`로 자동 변경 (v2 cleanup)

## 환경변수

`NOTION_DB_SEO_OPPORTUNITIES`
