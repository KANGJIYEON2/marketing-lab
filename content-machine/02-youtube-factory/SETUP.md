# 02 유튜브 팩토리 — 세팅 가이드

`workflows/youtube-factory.json`을 임포트해서 돌리기까지.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) 완료

**소요 시간**: 1~2시간

---

## 0. 준비물 체크

- [ ] Layer 0 완료 (n8n·Notion·Slack·Anthropic credentials)
- [ ] 모니터링할 **YouTube 채널 ID** (보통 본인 채널)
- [ ] **Transcript API 키** (3개 옵션 중 하나 선택, 아래 §2)
- [ ] Notion `ContentDrafts` DB에 본 자동화 속성이 모두 있는지 확인 ([스키마](../../docs/notion-schemas/content-drafts.md))

---

## 1. YouTube 채널 ID 확인

본인 채널 페이지 → URL이 두 가지 형식일 수 있다:

| URL 형식 | 채널 ID 얻는 법 |
|---|---|
| `youtube.com/channel/UCxxxxxxxxxxxxxxxxxxxxxx` | URL에 그대로 있음 |
| `youtube.com/@handle` | 채널 페이지 소스 보기 → `"channelId":"UC..."` 검색 |

`.env`에 등록:
```bash
YOUTUBE_CHANNEL_ID=UCxxxxxxxxxxxxxxxxxxxxxx
```

채널 ID는 `UC`로 시작하는 24자 문자열이어야 한다.

---

## 2. Transcript API 선택

YouTube 자체 API는 자막에 접근하려면 OAuth + 까다로운 권한이 필요하다. 실용적인 3가지 옵션:

### Option A: Supadata (권장, MVP 기본값)

- https://supadata.ai → 가입 → API key 발급
- 무료 tier: 월 100 영상 transcript
- 워크플로 default endpoint: `https://api.supadata.ai/v1/youtube/transcript`
- `.env`:
  ```bash
  SUPADATA_API_KEY=sk_...
  ```

### Option B: youtube-transcript-api (자체 호스팅)

Python 패키지 [`youtube-transcript-api`](https://github.com/jdepoix/youtube-transcript-api)를 작은 Flask 서버로 래핑해서 셀프 호스팅. 무료지만 IP 차단 가능성 있어 프록시 필요.

워크플로 `Fetch Transcript` 노드의 URL을 자체 서버로 교체.

### Option C: Apify Actor

Apify의 `youtube-transcript-scraper` actor 사용. 토큰 기반 과금 ($0.50/1k 영상 수준).

워크플로 `Fetch Transcript` 노드를 Apify Actor HTTP 호출로 교체:
```
POST https://api.apify.com/v2/acts/{actor_id}/run-sync-get-dataset-items?token={APIFY_TOKEN}
body: { "videoId": "{{ $('Parse RSS').item.json.video_id }}" }
```

### 선택 기준

| 옵션 | 시작 비용 | 운영 비용 | 안정성 |
|---|---|---|---|
| Supadata | 0 (무료) | $20/월~ | 高 |
| youtube-transcript-api | 호스팅비 | ~$5/월 | 中 (IP 이슈) |
| Apify | 0 | $5~10/월 | 高 |

**MVP는 Supadata로 시작**, 영상 수 늘면 Apify로 전환.

---

## 3. Notion `ContentDrafts` DB 확인

[스키마 문서](../../docs/notion-schemas/content-drafts.md) 기준 다음 속성이 있어야 함:

- `channel` (Select: `blog`, `x_thread`, `linkedin` 포함)
- `source_type` (Select: `youtube_video` 포함)
- `source_ref` (URL)
- `meta_desc` (Rich text)
- `target_keyword` (Select)
- `status` (Select: `review` 포함)

본 워크플로는 영상 1편당 **3개 행을 생성** (blog / x_thread / linkedin). 모두 `status=review`로 시작.

---

## 4. 워크플로 임포트

n8n UI → `...` → **Import from File** → `workflows/youtube-factory.json`

### 임포트 후 확인

- [ ] **모든 Claude 노드**의 credential에 `Anthropic API` 선택
- [ ] **Notion 노드 2개** (Already Processed? / Notion: Create Draft)의 credential 선택, `databaseId`가 `$env.NOTION_DB_CONTENT_DRAFTS` 참조 확인
- [ ] **Fetch Transcript** 노드의 API endpoint·헤더가 선택한 옵션과 일치
- [ ] **Slack** 노드 credential + 채널 (`$env.SLACK_CHANNEL_LOG`)
- [ ] **Config** 노드의 `brand_tone`을 본인 브랜드에 맞게 수정
- [ ] **Config** 노드의 `target_keyword_pool`이 01 아이디어 엔진의 키워드와 일치하는지 확인 (정렬되어야 ContentIdeas DB와 호환)

---

## 5. 첫 실행 (수동 테스트)

### 방법 A: 기존 영상으로 테스트

`Every 30 min` 트리거 노드 우클릭 → **Execute Node**. 그러면 채널의 최근 5개 영상이 후보로 들어옴.

이미 처리된 영상은 `Already Processed?` 노드에서 걸러짐 → 처음엔 모두 신규.

### 방법 B: 가짜 영상 ID로 단발 테스트

`Parse RSS` 노드를 비활성화하고, 그 자리에 임시 Set 노드로:
```json
{
  "video_id": "테스트할 영상 ID",
  "title": "테스트 제목",
  "url": "https://youtube.com/watch?v=...",
  "description": ""
}
```

실행 후 정상 작동 확인되면 임시 노드 제거하고 RSS 노드 복원.

### 각 단계 통과 시그널

| 노드 | 확인 |
|---|---|
| Fetch YouTube RSS | XML 응답 (`<feed>...</feed>`) |
| Parse RSS | `{video_id, title, url, ...}` 객체 5개 |
| Already Processed? | 신규 영상은 빈 결과 |
| Fetch Transcript | `content` 또는 `text` 필드에 자막 문자열 |
| Claude: Analyze Video | JSON `{core_message, chapters, ...}` |
| Claude (Blog/X/LinkedIn) | 각각 JSON 응답 |
| Build 3 Notion Rows | 정확히 3개 아이템 |
| Notion: Create Draft | DB에 3개 행 생성 (`status=review`) |
| Slack | `#automation-log`에 메시지 도착 |

---

## 6. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| `Parse RSS` 결과 0개 | 채널 ID 오타 또는 채널 비공개 | YouTube에서 직접 RSS URL 확인: `youtube.com/feeds/videos.xml?channel_id=...` |
| Transcript API 4xx | 자막이 없는 영상 (커뮤니티 영상, 라이브) | MVP에서는 자막 있는 영상만. 감지 후 fallback 분기 추가는 v2 |
| Claude `Invalid JSON` | 출력에 markdown fence | `Bundle Context` Code 노드의 정규식이 잡아냄. 그래도 실패 시 프롬프트 끝에 "No prose, no markdown, no code fences." 강조 |
| Notion `property does not exist` | `ContentDrafts` 스키마 불일치 | [스키마 문서](../../docs/notion-schemas/content-drafts.md)와 정확히 일치 확인. 영문·snake_case |
| 같은 영상 중복 처리 | `Already Processed?` 필터 작동 안 함 | Notion 노드 filter의 `source_ref|url contains video_id` 확인. RSS URL에 video_id가 포함됨 |
| LLM이 영어로만 출력 | 시스템 프롬프트 "match transcript language" 무시 | 프롬프트 마지막에 `Important: Use Korean if transcript is in Korean.` 추가 |

---

## 7. 활성화

수동 테스트 통과 후 워크플로 토글을 **Active**로.

기본 30분 주기. 영상 업로드 후 30분 이내에 자동 처리.

---

## 8. 운영 팁

### 검수 워크플로

1. 영상 업로드 → 30분 후 Slack 알림
2. Notion `ContentDrafts` DB에서 `status=review` 필터로 신규 3건 확인
3. 각 행 본문 검토 → 수정 → `status=approved`로 변경
4. 발행: 현재 MVP는 **수동** (블로그 시스템·X·LinkedIn에 복붙)

### v2 추가 후보

PRD `MVP 범위 (제외)` 섹션 항목 중:
1. **쇼츠 스크립트 자동 생성** — 새 프롬프트 `shorts-script-from-video.md`
2. **자동 발행** — WordPress REST / X API / LinkedIn API 노드 추가, Slack 승인 후 트리거
3. **자막 없는 영상 지원** — Whisper API 분기

각각 별도 PR로 진행.

### 비용 추정 (영상 1편당)

| 단계 | LLM 토큰 | 비용 |
|---|---|---|
| analyze-video | 5k in + 0.8k out | $0.03 |
| blog-from-video | 9k in + 2.5k out | $0.07 |
| x-thread-from-video | 2k in + 1k out | $0.02 |
| linkedin-from-video | 1.5k in + 0.8k out | $0.015 |
| **합계** | | **≈ $0.14 / 영상** |

주 1편 = 월 ~$0.60, 주 3편 = 월 ~$2. transcript API 비용 별도.

---

## 9. 다음 단계

- 첫 1주 검수율(`status=review → approved` 비율) 추적
- 검수율 < 60% → 브랜드 톤 프롬프트 보강
- 검수율 > 80% → v2 자동 발행 단계로 진행
- 영상이 쌓이면 [05 SEO 헌터](../05-seo-hunter/) 진입 (발행된 블로그 글의 GSC 데이터 활용)
