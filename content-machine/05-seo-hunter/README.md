# 05 · SEO 11~20위 기회 헌터

> Google Search Console에서 2페이지 키워드 자동 발굴 → 기존 글 업데이트 액션

**카테고리**: [content-machine](../README.md) · **로드맵**: Week 5 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/seo-action-items.md` — 구현 시 추가
- `workflows/seo-hunter.json` — n8n 임포트용 (구현 시 추가)

## 폴더 구조

```
05-seo-hunter/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 발행된 글 10편 이상, GSC 등록
- **활용**: 발행된 [`ContentDrafts`](../../docs/notion-schemas/content-drafts.md)와 매칭
- **연계**: 09 주간 리포트에서 SEO 액션 진행률 추적
