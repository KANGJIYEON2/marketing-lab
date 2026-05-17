# 04 · 경쟁사 인텔리전스 봇

> 경쟁사 블로그·SNS·광고·가격 페이지 변화 일간 모니터링 → 변경 시 Slack 알림

**카테고리**: [ops-reduction](../README.md) · **로드맵**: Week 4 · **상태**: MVP 구현 완료

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — 경쟁사 Config 채우기·Meta Ad Library·임포트
- [`prompts/competitor-change-analyze.md`](./prompts/competitor-change-analyze.md) — 분석 + 영향도 + 액션 단일 호출
- [`prompts/competitor-action-suggest.md`](./prompts/competitor-action-suggest.md) — 액션 품질 가이드 (메타)
- [`workflows/competitor-intel.json`](./workflows/competitor-intel.json) — n8n 임포트용

## 폴더 구조

```
04-competitor-intel/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md) + [CompetitorTimeline DB](../../docs/notion-schemas/competitor-timeline.md), `META_GRAPH_ACCESS_TOKEN` (Meta Ad Library 사용 시)
- **시너지**: 06이 경쟁사 광고 패턴 참고, 01이 경쟁사 인기 토픽 시드 활용
- **연계**: 09 주간 리포트가 high-impact change 집계
