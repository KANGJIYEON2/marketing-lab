# ContentIdeas DB

01 아이디어 엔진이 수집한 글감을 저장하는 단일 SoT.

**사용 자동화**: 01(쓰기), 02(읽기 — 영상 기획 시드), 05(읽기 — 기존 글과 매칭)

## 속성

| 속성 이름 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `Name` | Title | ✓ | 글감 제목 (보통 LLM이 제안한 첫 번째 title) |
| `pain_point` | Rich text | ✓ | 페인포인트 한 줄 |
| `source` | Select | ✓ | `reddit`, `x`, `youtube`, `naver` |
| `source_url` | URL | ✓ | 원본 링크 |
| `keyword` | Select | ✓ | 트래킹 키워드 (Config 노드 키워드 + `other`) |
| `score` | Number | ✓ | LLM 점수 0-10 |
| `engagement` | Number | | 원본 인게이지먼트 |
| `status` | Select | ✓ | `new`, `triaged`, `queued`, `drafted`, `published`, `archived` |
| `assigned_to` | Person | | 작업 담당 (선택) |
| `created_at` | Created time | (자동) | |
| `last_edited` | Last edited time | (자동) | |

## Select 옵션 사전 정의

### `source` 옵션
- `reddit` (orange)
- `x` (black)
- `youtube` (red)
- `naver` (green)

### `keyword` 옵션
초기에는 비워두고 자동으로 채워지게 두거나, Config 노드 키워드를 미리 추가:
- `n8n automation`
- `marketing automation`
- `solo founder`
- `saas growth`
- `ai workflow`
- `other`

### `status` 옵션
- `new` (default, blue) — 방금 수집됨
- `triaged` (yellow) — 사람이 검토함
- `queued` (purple) — 작성 대기열
- `drafted` (orange) — 초안 작성 중
- `published` (green) — 발행 완료
- `archived` (gray) — 폐기

## 뷰 권장

- **Triage queue**: filter `status = new`, sort `score desc`
- **This week's queue**: filter `status = queued`
- **Keyword performance**: group by `keyword`, count
- **Source mix**: group by `source`, last 7 days

## 본문 (페이지 콘텐츠)

01 워크플로가 페이지 본문에 자동으로 추가:
- `H2 Suggested Titles` + bullet list (titles 3개)
- `H2 Source Quote` + quote block (원본 발췌 500자)

## 환경변수

DB ID는 `.env`의 `NOTION_DB_CONTENT_IDEAS`로 참조.
