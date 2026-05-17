# 02 · 유튜브 → 멀티채널 콘텐츠 팩토리

> 영상 1편 업로드 → 블로그·쇼츠·X·LinkedIn·뉴스레터 자동 생성

**카테고리**: [content-machine](../README.md) · **로드맵**: Week 3 · **상태**: MVP 구현 완료

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — 임포트·credentials·첫 실행 절차
- [`prompts/analyze-video.md`](./prompts/analyze-video.md) — 영상 분석 (공통 컨텍스트 추출)
- [`prompts/blog-from-video.md`](./prompts/blog-from-video.md) — 블로그 글 생성
- [`prompts/x-thread-from-video.md`](./prompts/x-thread-from-video.md) — X 스레드 생성
- [`prompts/linkedin-from-video.md`](./prompts/linkedin-from-video.md) — LinkedIn 포스트 생성
- [`workflows/youtube-factory.json`](./workflows/youtube-factory.json) — n8n 임포트용

## 폴더 구조

```
02-youtube-factory/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 유튜브 채널 운영 중
- **활용**: 01 [`ContentIdeas`](../../docs/notion-schemas/content-ideas.md)에서 영상 주제 시드
- **후행**: 06(광고)이 02의 [`ContentDrafts`](../../docs/notion-schemas/content-drafts.md)를 카피 시드로 사용
