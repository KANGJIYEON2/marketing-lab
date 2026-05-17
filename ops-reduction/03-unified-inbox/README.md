# 03 · 댓글·DM·문의 통합 인박스

> 모든 채널 댓글·DM → Slack 통합 + AI 분류·답변 초안

**카테고리**: [ops-reduction](../README.md) · **로드맵**: Week 3 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/` — `inbox-classify-reply.md`, `tone-guide-by-channel.md`
- `workflows/unified-inbox.json` — n8n 임포트용 (구현 시 추가)

## 폴더 구조

```
03-unified-inbox/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 각 채널 API 토큰
- **독립 실행 가능** — 다른 자동화 의존 없음
