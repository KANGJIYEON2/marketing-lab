# CompetitorTimeline DB

04 경쟁사 인텔이 발견한 변경사항과 일간 스냅샷을 시계열로 적재. 두 가지 행 타입(`snapshot`과 `change`)이 공존.

**사용 자동화**: 04(쓰기), 06(읽기 — 경쟁사 광고 패턴 참고), 09(읽기 — 분기별 경쟁 동향 리포트)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | `{competitor}: {source} — {summary 30자}` |
| `competitor` | Select | ✓ | 경쟁사 이름 (Config에 정의된 이름) |
| `source` | Select | ✓ | `blog`, `pricing`, `meta_ads`, `social`, `producthunt` |
| `entry_type` | Select | ✓ | `snapshot`, `change` |
| `source_url` | URL | | 원본 URL |
| `external_id` | Rich text | | RSS guid·광고 ID 등 (dedup용) |
| `captured_at` | Date | ✓ | 수집 시각 |
| `summary` | Rich text | | 변경 요약 (entry_type=change) 또는 스냅샷 설명 |
| `category` | Select | | `pricing`, `positioning`, `feature`, `content`, `launch`, `other` |
| `impact_score` | Number | | 우리에게 미치는 영향 0-10 |
| `our_action` | Rich text | | Claude가 제안한 우리 액션 (change only) |
| `raw_payload` | Rich text | | 본문 또는 페이지 콘텐츠로 |
| `snapshot_hash` | Rich text | | HTML 본문 SHA-256 앞 16자 (snapshot용) |

## Select 옵션

### `competitor`
초기에는 비워둠. Config에 정의된 이름이 자동으로 추가됨. 권장: `Competitor A`, `Competitor B`, `Competitor C` 같은 단순 라벨 (실제 이름 노출이 부담스러우면).

### `source`
- `blog` (gray)
- `pricing` (red)
- `meta_ads` (blue)
- `social` (pink) — v2
- `producthunt` (orange) — v2

### `entry_type`
- `snapshot` (gray) — 매일 저장하는 스냅샷 (diff 비교용)
- `change` (yellow) — 의미 있는 변경 감지 시

### `category`
- `pricing` (red) — 가격 인상/할인/플랜 변경
- `positioning` (purple) — 메인 카피·USP 변경
- `feature` (blue) — 신기능·제품 변경
- `content` (gray) — 블로그 새 글
- `launch` (orange) — 신제품·캠페인 런칭
- `other` (gray)

## 뷰 권장

- **Changes only (timeline)**: filter `entry_type = change`, sort `captured_at desc`
- **High impact**: filter `entry_type = change AND impact_score >= 7`
- **By competitor**: group `competitor`, filter `entry_type = change`
- **Pricing watch**: filter `source = pricing AND entry_type = change`
- **Latest snapshots**: filter `entry_type = snapshot`, sort `captured_at desc`, grouped by `competitor + source`

## Diff 로직 (04 워크플로 내부)

매 실행 시 source별로:
1. 현재 HTML/RSS/Ad 응답을 받음
2. Notion에서 `competitor + source + entry_type=snapshot`의 가장 최근 행 fetch
3. `snapshot_hash` 비교
4. 동일 → 신규 행 생성 안 함 (기존 snapshot의 `captured_at`만 업데이트, v2)
5. 다름 → Claude 분석 후 `entry_type=change` 신규 행 + 새 `snapshot` 행 모두 추가

## 데이터 수명 정책

- `snapshot` 행: 같은 (competitor, source) 조합에서 가장 최근 1건만 유지. 나머지는 30일 후 삭제 (수동 또는 v2 cleanup 워크플로)
- `change` 행: 영구 보관 (히스토리 가치)

## 환경변수

`NOTION_DB_COMPETITOR_TIMELINE`
