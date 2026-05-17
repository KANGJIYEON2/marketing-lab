# ContentDrafts DB

02 유튜브 팩토리, 05 SEO 헌터 등이 생성한 콘텐츠 초안과 발행 상태를 추적.

**사용 자동화**: 02(쓰기), 05(쓰기 업데이트 액션), 06(읽기 — 광고 카피 시드), 08(쓰기 UGC 가공 콘텐츠)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | 콘텐츠 제목 |
| `channel` | Select | ✓ | `blog`, `x_thread`, `linkedin`, `instagram`, `newsletter`, `youtube_short` |
| `source_type` | Select | ✓ | `youtube_video`, `seo_update`, `ugc`, `manual`, `idea` |
| `source_ref` | URL | | 원본 (영상 URL, 기존 글 URL 등) |
| `idea_id` | Relation | | `ContentIdeas` 연결 (선택) |
| `body` | Rich text | ✓ | 본문 (1900자 한도 시 페이지 콘텐츠 사용) |
| `meta_title` | Rich text | | SEO 메타 타이틀 (blog only) |
| `meta_desc` | Rich text | | SEO 메타 디스크립션 |
| `target_keyword` | Select | | 타겟 키워드 |
| `status` | Select | ✓ | `draft`, `review`, `approved`, `scheduled`, `published`, `archived` |
| `scheduled_at` | Date | | 예약 발행 시간 |
| `published_at` | Date | | 실제 발행 시간 |
| `published_url` | URL | | 발행된 URL |
| `views` | Number | | 발행 후 측정값 (09가 업데이트) |
| `engagement` | Number | | likes+comments+shares |
| `created_at` | Created time | (자동) | |

## Select 옵션

### `channel`
- `blog` (gray)
- `x_thread` (black)
- `linkedin` (blue)
- `instagram` (pink)
- `newsletter` (yellow)
- `youtube_short` (red)

### `source_type`
- `youtube_video` (red) — 02가 생성
- `seo_update` (green) — 05의 업데이트 액션
- `ugc` (purple) — 08이 가공
- `manual` (gray) — 사람이 직접
- `idea` (orange) — 01 ContentIdeas에서 직접 발전

### `status`
- `draft` (gray)
- `review` (yellow) — 사람 검수 대기
- `approved` (orange) — 검수 통과, 발행 큐
- `scheduled` (purple) — 예약됨
- `published` (green)
- `archived` (gray)

## 뷰 권장

- **Review queue**: filter `status = review`
- **This week scheduled**: filter `status = scheduled`, sort `scheduled_at asc`
- **By channel**: group `channel`
- **Performance**: filter `status = published`, sort `views desc`

## 환경변수

`NOTION_DB_CONTENT_DRAFTS`
