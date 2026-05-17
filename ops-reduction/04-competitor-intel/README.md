# 04 · 경쟁사 인텔리전스 봇

> 경쟁사 블로그·SNS·광고·가격 페이지 변화 일간 모니터링 → 변경 시 Slack 알림

**카테고리**: [ops-reduction](../README.md) · **로드맵**: Week 4 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/` — `competitor-change-analyze.md`, `competitor-action-suggest.md`
- `workflows/competitor-intel.json` — n8n 임포트용 (구현 시 추가)

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

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), `.env`의 `COMPETITOR_DOMAINS`
- **시너지**: 06이 경쟁사 광고 패턴 참고, 01이 경쟁사 인기 토픽 시드 활용
