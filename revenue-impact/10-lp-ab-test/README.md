# 10 · 랜딩페이지 A/B 자동 실험

> 헤드라인·CTA 변형 자동 생성 → 배포 → 위너 자동 채택

**카테고리**: [revenue-impact](../README.md) · **로드맵**: Week 7 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/` — `lp-variant-headlines.md`, `lp-variant-cta.md`, `lp-variant-social-proof.md`
- `workflows/lp-ab-test.json` — n8n 임포트용 (구현 시 추가)

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

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 일평균 LP 트래픽 100+
- **저장소**: [`CampaignMetrics`](../../docs/notion-schemas/campaign-metrics.md) DB
- **시너지**: 07(리드 시퀀스)이 전환 측정, 06(광고)이 트래픽 공급
