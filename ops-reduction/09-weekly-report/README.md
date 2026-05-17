# 09 · 주간 마케팅 리포트 + 다음주 액션 추천

> 일요일 밤 KPI 자동 집계 → 월요일 9시 Slack 인사이트 + 액션 3개

**카테고리**: [ops-reduction](../README.md) · **로드맵**: Week 6 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/` — `weekly-insights-actions.md`, `kpi-trend-detect.md`
- `workflows/weekly-report.json` — n8n 임포트용 (구현 시 추가)

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
- **읽음**: 다른 자동화의 결과 ([`CampaignMetrics`](../../docs/notion-schemas/campaign-metrics.md), [`Leads`](../../docs/notion-schemas/leads.md), [`ContentDrafts`](../../docs/notion-schemas/content-drafts.md))
- **권장**: 다른 자동화가 일정 기간 데이터 쌓은 뒤 (Week 6 이후)
