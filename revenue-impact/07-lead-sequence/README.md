# 07 · 리드 캡처 → 개인화 환영 시퀀스

> 폼 제출 → enrich → 회사 상황에 맞춘 환영 메일 3단계

**카테고리**: [revenue-impact](../README.md) · **로드맵**: Week 4 · **상태**: MVP 구현 완료 (Day 0 단일 메일)

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — Resend 도메인 인증·Apollo·폼 webhook·테스트
- [`prompts/lead-score-classify.md`](./prompts/lead-score-classify.md) — 스코어 + 페르소나 + 핫시그널
- [`prompts/welcome-email-personalized.md`](./prompts/welcome-email-personalized.md) — Day 0 환영 메일
- [`prompts/case-study-pick.md`](./prompts/case-study-pick.md) — Day 2 케이스 선택 (v2 명세)
- [`workflows/lead-sequence.json`](./workflows/lead-sequence.json) — n8n 임포트용

## 폴더 구조

```
07-lead-sequence/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), Apollo/Clearbit·Resend 계정
- **저장소**: [`Leads`](../../docs/notion-schemas/leads.md) DB
- **시너지**: 10(LP A/B)이 폼 트래픽 공급
