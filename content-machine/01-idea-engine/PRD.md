# 01. 콘텐츠 아이디어 엔진

> 타겟 고객의 **진짜 페인포인트**를 매일 자동 수집 → 글감 큐로 정리

## 1. 페인포인트

- "오늘 뭐 쓰지?"가 매일 30~60분을 잡아먹음
- 키워드 도구는 검색량은 알려주지만, **사람들이 진짜 어떤 표현으로 고민하는지**는 모름
- 트렌드 따라잡기 = 수동으로 매일 Reddit·X·유튜브 댓글 뒤지기

## 2. KPI

| 지표 | 목표 |
|---|---|
| 주간 신규 글감 | 50개+ |
| 글감 → 발행 전환률 | 20%+ (10개/주 발행) |
| 페인포인트 1개당 수집 시간 | < 30초 (자동) |

## 3. Input / Output

**Input**
- 타겟 키워드 (예: "n8n 자동화", "1인 창업", "SaaS 마케팅")
- 모니터링 소스: Reddit subreddit 리스트, X 키워드, 유튜브 채널, 네이버 카페

**Output**
- Notion DB `ContentIdeas`에 행 추가
  - `title`, `pain_point`, `source_url`, `quote`, `keyword`, `score`, `status`(new/triaged/queued/published)

## 4. n8n 워크플로

```
[Cron: 매일 오전 7시]
  ├─▶ Reddit Search (subreddit 리스트 × 키워드)
  ├─▶ X Search API (키워드 + min_engagement)
  ├─▶ YouTube Comments (지정 채널 신규 영상)
  └─▶ 네이버 카페 (스크래퍼 또는 외부 API)
        │
        ▼
  [Merge] → 중복 제거 (URL 해시)
        │
        ▼
  [Claude API] 각 댓글/포스트에 대해:
    - 페인포인트 한 줄 추출
    - 잠재 글 제목 3개 제안
    - 타겟 키워드 매칭도 0-10
        │
        ▼
  [Filter] score >= 6
        │
        ▼
  [Notion: Append to ContentIdeas]
        │
        ▼
  [Slack #content-ideas] 상위 5개 요약 전송
```

## 5. 필요한 통합

- Reddit API (무료, OAuth)
- X API v2 (basic tier $200/월 또는 Apify 스크래퍼 $)
- YouTube Data API (무료 쿼터)
- 네이버 카페: 공식 API 없음 → Apify 또는 자체 스크래퍼
- Claude API (Sonnet 4.6 권장, 토큰 효율)
- Notion API
- Slack webhook

## 6. MVP 범위 (Week 1)

**포함**
- Reddit + X 만 (네이버/유튜브는 v2)
- 키워드 3~5개로 시작
- Notion 단일 DB 적재
- Slack 다이제스트

**제외 (v2)**
- 네이버 카페·유튜브 댓글
- 자동 글감 → 글 초안 생성 (M2 별도)
- 트렌드 시계열 분석

## 7. 의존성

- **선행**: Layer 0 인프라 (Notion DB, n8n credentials)
- **후행**: 02(유튜브 팩토리), 05(SEO 헌터)가 이 DB 활용

## 8. 리스크

- X API 비용 → 시작은 Apify 스크래퍼 검토
- LLM 토큰 비용 → 1차 필터(키워드 매칭)로 LLM 호출 최소화
- 노이즈 → 페인포인트 점수 6점 이상만 채택, 주간 리뷰로 임계값 튜닝

## 9. 프롬프트

`prompts/extract-painpoint.md` (구현 시 작성)
- input: raw 댓글/포스트
- output: JSON `{pain_point, titles[], keyword_match, score}`
