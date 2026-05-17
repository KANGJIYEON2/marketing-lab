# UGCLibrary DB

08 UGC 컬렉터가 수집한 외부 멘션·리뷰·태그를 라이브러리화. 허락 받은 UGC는 광고·LP·SNS 소재로 재활용 (가공은 v2).

**사용 자동화**: 08(쓰기), 06(읽기 — 광고 카피 시드), 10(읽기 — LP 후기 섹션 v2)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | `{author} on {channel} — {content 30자}` |
| `channel` | Select | ✓ | `instagram_mention`, `instagram_tag`, `instagram_hashtag`, `x_mention`, `naver_blog`, `review_site`, `manual` |
| `author` | Rich text | ✓ | 작성자 표시명 또는 핸들 |
| `author_handle` | Rich text | | @username 등 외부 식별자 |
| `content` | Rich text | ✓ | 원본 본문 |
| `pull_quote` | Rich text | | 활용 가능한 핵심 인용구 (Claude 추출) |
| `media_url` | URL | | 이미지·영상 URL (있으면) |
| `source_url` | URL | ✓ | 원본 게시물 링크 |
| `external_id` | Rich text | ✓ | dedup용 (post ID / mention ID) |
| `captured_at` | Date | ✓ | 수집 시간 |
| `posted_at` | Date | | 원본 게시 시간 |
| `sentiment` | Select | ✓ | `positive`, `neutral`, `negative` |
| `usability_score` | Number | ✓ | 활용 가능성 0-10 |
| `usage_formats` | Multi-select | | `carousel`, `ad_creative`, `lp_testimonial`, `repost`, `case_study` |
| `permission_status` | Select | ✓ | `needs_permission`, `permission_requested`, `approved`, `denied`, `not_required` |
| `permission_proof` | URL | | 허락 받은 메시지 캡처 URL (자체 저장소) |
| `used_in` | Rich text | | 어디에 사용했는지 (광고 ID, LP URL 등) |

## Select 옵션

### `channel`
- `instagram_mention` (pink) — @브랜드 멘션
- `instagram_tag` (pink) — 사진에 브랜드 태그
- `instagram_hashtag` (pink) — 브랜드 해시태그
- `x_mention` (black) — v2
- `naver_blog` (green) — v2
- `review_site` (yellow) — v2
- `manual` (gray) — 사람이 직접 추가

### `sentiment`
- `positive` (green)
- `neutral` (gray)
- `negative` (red) — 자산화 X, 별도 운영팀 라우팅

### `usage_formats` (Multi-select)
- `carousel` — IG 캐러셀 슬라이드 소스
- `ad_creative` — 광고 소재 (text overlay 또는 인용)
- `lp_testimonial` — 랜딩페이지 후기 섹션
- `repost` — 그대로 SNS 리포스트 (스토리)
- `case_study` — 케이스 스터디 글 시드

### `permission_status`
- `needs_permission` (yellow) — 새로 수집, 허락 미요청
- `permission_requested` (orange) — DM/이메일로 요청함, 답변 대기
- `approved` (green) — 사용 허락 받음
- `denied` (red) — 거절 (영구 보관, 절대 사용 X)
- `not_required` (gray) — 공개 리뷰 등 인용 가능

## 뷰 권장

- **Triage (needs permission)**: filter `permission_status = needs_permission AND sentiment = positive`, sort `usability_score desc`
- **Ready to use**: filter `permission_status in [approved, not_required]`
- **Awaiting permission**: filter `permission_status = permission_requested`
- **Negative (route to ops)**: filter `sentiment = negative` — 03 통합 인박스로 가야 할 케이스
- **By channel**: group `channel`

## 본문 (페이지 콘텐츠)

워크플로가 페이지 본문에 자동 추가:
- `H2 Original Content` + 본문
- `H2 Pull Quote` + 인용 (있으면)
- `Image` block — `media_url`이 있으면

## 데이터 수명 정책

- `permission_status = denied` 행: 영구 보관 (재요청 방지)
- `sentiment = negative` 행: 30일 후 archive (필요 시 사람이 운영팀으로 이관)
- 그 외: 영구 보관 (자산)

## 허락 받기 워크플로

본 자동화는 **수집만**. 허락 요청은 사람이 수동으로:
1. `needs_permission` 행 검토
2. DM/댓글로 사용 허락 요청 ("저희 브랜드 채널/광고에 이 후기 인용해도 될까요?")
3. 답변 받으면 `permission_status` 수동 업데이트
4. 캡처 URL을 `permission_proof`에 저장

v2 자동화 후보:
- 표준 허락 요청 메시지 자동 발송 (Meta Graph API)
- 답변 자동 감지 → status 업데이트

## 환경변수

`NOTION_DB_UGC_LIBRARY`
