# 05. SEO 11~20위 기회 헌터

> Google Search Console에서 **2페이지에 박힌 키워드** 자동 발굴 → 기존 글 업데이트 액션 생성

## 1. 페인포인트

- 신규 글보다 **기존 글 업데이트가 ROI 10배 높음** (이미 색인됨, 권위 있음)
- 어떤 글이 11~20위에 있는지 매번 GSC 뒤지기 번거로움
- 1페이지 진입에 필요한 보강 포인트를 사람이 분석하기엔 글이 너무 많음

## 2. KPI

| 지표 | 목표 |
|---|---|
| 주간 발굴 기회 키워드 | 20개+ |
| 액션 → 1페이지 진입률 | 30%+ (4주 내) |
| 업데이트 글 1편당 추가 클릭 | 50+ /월 |

## 3. Input / Output

**Input**
- Google Search Console API (사이트 등록 필요)
- 기간: 직전 28일
- 필터: position 11~20, impressions >= 100

**Output**
- Notion `SEOOpportunities` DB
  - `url`, `keyword`, `position`, `impressions`, `ctr`, `action_items[]`, `priority`, `status`

## 4. n8n 워크플로

```
[Cron: 매주 월요일 6시]
        │
        ▼
[GSC API: Search Analytics]
  - 28일, position 11-20, impressions >= 100
  - 그룹: URL × Query
        │
        ▼
[Filter] 동일 URL에 가장 많은 impressions 키워드만
        │
        ▼
[For each: 각 (URL, keyword)]
  ├─▶ Fetch: 해당 URL HTML
  ├─▶ SERP API: 현재 1페이지 결과 (Top 10) 페이지 본문 요약
  └─▶ Claude API:
      "내 글 vs Top 10 결과 비교.
       1페이지 진입에 필요한 보강 포인트 3~5개 액션 아이템으로."
        │
        ▼
[Notion: SEOOpportunities 적재]
  - priority = impressions × (avg_top10_ctr - current_ctr)
        │
        ▼
[Slack 다이제스트] "이번주 SEO 기회 Top 5: [Notion 링크]"
```

## 5. 필요한 통합

- Google Search Console API (서비스 계정)
- SERP API (Serpapi, DataForSEO, 또는 ScraperAPI)
- Claude API
- Notion API
- Slack webhook

## 6. MVP 범위 (Week 3~4)

**포함**
- GSC → 기회 키워드 추출 + Notion 적재
- Claude로 액션 아이템 생성
- Slack 주간 다이제스트

**제외 (v2)**
- 자동으로 글 본문 업데이트 (사람 검수 단계 유지)
- 백링크 분석
- 경쟁 글 클러스터 분석

## 7. 의존성

- **선행**: 발행된 글이 일정량 있어야 의미 있음 (10편+)
- **시너지**: 09(주간 리포트)에서 SEO 액션 진행률 추적

## 8. 리스크

- GSC 데이터 지연 (2~3일) → 충분
- SERP API 비용 → 우선순위 상위만 SERP 호출
- 액션의 품질 → 첫 4주 사람 검수율 높게 유지하며 프롬프트 튜닝

## 9. 프롬프트

`prompts/seo-action-items.md`
- input: 내 글 본문 + Top 10 요약 + 키워드
- output: JSON `{missing_sections[], weak_points[], recommended_additions[]}`
