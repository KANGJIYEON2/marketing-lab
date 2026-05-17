# 10 · 랜딩페이지 A/B 자동 실험

> 헤드라인·CTA 변형 자동 생성 → 배포 → 위너 자동 채택

**카테고리**: [revenue-impact](../README.md) · **로드맵**: Week 7 · **상태**: MVP 구현 완료 (헤드라인 A/B, CTA·Social proof는 v2)

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — GA4 custom dimension, LP 플랫폼별 배포, z-test 유의성
- [`prompts/lp-variant-headlines.md`](./prompts/lp-variant-headlines.md) — 5종 후킹 각도, MVP 사용
- [`prompts/lp-variant-cta.md`](./prompts/lp-variant-cta.md) — v2 명세 (commitment level 3단계)
- [`prompts/lp-variant-social-proof.md`](./prompts/lp-variant-social-proof.md) — v2 명세 (UGC 연동)
- [`workflows/generate-variants.json`](./workflows/generate-variants.json) — webhook 실험 시작
- [`workflows/check-results.json`](./workflows/check-results.json) — daily cron 유의성 측정

## 폴더 구조

```
10-lp-ab-test/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 일평균 LP 트래픽 100+, GA4 custom dimension 등록 가능
- **저장소**: [`CampaignMetrics`](../../docs/notion-schemas/campaign-metrics.md) DB (entity_type=lp_variant + experiment_id 그룹화)
- **시너지**: 07(리드 시퀀스)이 전환 측정, 06(광고)이 트래픽 공급, 08(UGC)이 social proof v2 자산
- **연계**: 09 주간 리포트가 활성 실험 + 위너 trend 집계
