# 01 · 콘텐츠 아이디어 엔진

> Reddit·X 페인포인트 자동 수집 → 글감 큐 (Notion `ContentIdeas`)

**카테고리**: [content-machine](../README.md) · **로드맵**: Week 2 · **상태**: MVP 구현 완료

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — 임포트·credentials·첫 실행 절차
- [`prompts/extract-painpoint.md`](./prompts/extract-painpoint.md) — LLM 시스템 프롬프트
- [`prompts/daily-digest.md`](./prompts/daily-digest.md) — Slack 다이제스트 포맷
- [`workflows/idea-engine.json`](./workflows/idea-engine.json) — n8n 임포트용

## 폴더 구조

```
01-idea-engine/
├── README.md       # 이 파일
├── PRD.md          # 제품 요구사항
├── SETUP.md        # 셋업 절차
├── prompts/        # LLM 프롬프트
├── workflows/      # n8n JSON
├── scripts/        # 보조 스크립트 (현재 없음)
└── docs/           # 추가 문서 (현재 없음)
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md)
- **후행**: 02, 05가 이 DB(`ContentIdeas`)를 input으로 사용
