# 03 · 댓글·DM·문의 통합 인박스

> 모든 채널 댓글·DM → Slack 통합 + AI 분류·답변 초안

**카테고리**: [ops-reduction](../README.md) · **로드맵**: Week 3 · **상태**: MVP 구현 완료

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — Meta App·Gmail OAuth·임포트·테스트 절차
- [`prompts/inbox-classify-reply.md`](./prompts/inbox-classify-reply.md) — 분류 + 답변 초안 한 번에
- [`prompts/tone-guide-by-channel.md`](./prompts/tone-guide-by-channel.md) — 채널별 톤 가이드
- [`workflows/unified-inbox.json`](./workflows/unified-inbox.json) — n8n 임포트용

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

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md) + [Inbox DB](../../docs/notion-schemas/inbox.md), Meta Graph 토큰, Gmail OAuth
- **독립 실행 가능** — 다른 자동화 의존 없음
- **연계**: 09 주간 리포트에서 응답 시간·누락률 KPI 집계
