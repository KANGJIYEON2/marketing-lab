# 06 · 광고 카피 진화 엔진

> 주간 광고 성과 → 잘 된 패턴 추출 → 신규 변형 10개 자동 생성·업로드

**카테고리**: [revenue-impact](../README.md) · **로드맵**: Week 5 · **상태**: MVP 구현 완료 (Meta 카피만, 자동 업로드 v2)

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — Meta Marketing API 권한, ad_name 명명 규칙, 운영 흐름
- [`prompts/ad-pattern-extract.md`](./prompts/ad-pattern-extract.md) — Top/Bottom 20% → 패턴 추출
- [`prompts/ad-variant-generate.md`](./prompts/ad-variant-generate.md) — 패턴 + 시드 → 10개 변형
- [`prompts/ad-guardrails.md`](./prompts/ad-guardrails.md) — Meta 정책 + 자사 브랜드 룰
- [`workflows/ad-evolution.json`](./workflows/ad-evolution.json) — n8n 임포트용

## 폴더 구조

```
06-ad-evolution/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), Meta 광고 운영 중, `META_GRAPH_ACCESS_TOKEN`에 `ads_read` 권한
- **저장소**: [`ContentDrafts`](../../docs/notion-schemas/content-drafts.md) (변형) + [`CampaignMetrics`](../../docs/notion-schemas/campaign-metrics.md) (성과)
- **활용**: 02 콘텐츠 시드 (블로그 글) — 변형 생성 영감
- **연계**: 09 주간 리포트가 CampaignMetrics에서 광고 트렌드 집계
