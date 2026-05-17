# 06 · 광고 카피 진화 엔진

> 주간 광고 성과 → 잘 된 패턴 추출 → 신규 변형 10개 자동 생성·업로드

**카테고리**: [revenue-impact](../README.md) · **로드맵**: Week 5 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/` — `ad-pattern-extract.md`, `ad-variant-generate.md`, `ad-guardrails.md`
- `workflows/ad-evolution.json` — n8n 임포트용 (구현 시 추가)

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

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 광고 운영 중 (월 광고비 $500+ 권장)
- **활용**: 02 콘텐츠, 08 UGC를 카피 시드로
- **연계**: 09 주간 리포트가 광고 성과 트래킹
