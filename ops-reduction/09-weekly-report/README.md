# 09 · 주간 마케팅 리포트 + 다음주 액션 추천

> 일요일 밤 KPI 자동 집계 → 월요일 9시 Slack 인사이트 + 액션 3개

**카테고리**: [ops-reduction](../README.md) · **로드맵**: Week 6 · **상태**: MVP 구현 완료 (트렌드 감지는 v2)

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — 다른 자동화 의존성, API 권한 통합, 운영 흐름
- [`prompts/weekly-insights-actions.md`](./prompts/weekly-insights-actions.md) — 인사이트 3 + 액션 3 단일 호출
- [`prompts/kpi-trend-detect.md`](./prompts/kpi-trend-detect.md) — v2 트렌드/이상치 감지 명세
- [`workflows/weekly-report.json`](./workflows/weekly-report.json) — n8n 임포트용

## 폴더 구조

```
09-weekly-report/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), GA4·Meta Ads·Resend 등 KPI 소스
- **읽음**: 다른 모든 자동화의 결과 — [`Leads`](../../docs/notion-schemas/leads.md), [`ContentDrafts`](../../docs/notion-schemas/content-drafts.md), [`Inbox`](../../docs/notion-schemas/inbox.md), [`CompetitorTimeline`](../../docs/notion-schemas/competitor-timeline.md), [`SEOOpportunities`](../../docs/notion-schemas/seo-opportunities.md), [`CampaignMetrics`](../../docs/notion-schemas/campaign-metrics.md)
- **쓰기**: CampaignMetrics에 weekly snapshot 아카이브 (v2 트렌드 분석용)
- **권장**: 다른 자동화가 최소 2~3주 운영된 뒤
