# 08 · UGC·리뷰 컬렉터

> 외부 리뷰·태그·멘션 자동 수집 → 캐러셀·광고·LP 소재화

**카테고리**: [content-machine](../README.md) · **로드맵**: Week 7 · **상태**: MVP 구현 완료 (수집만, 자동 가공은 v2)

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- [`SETUP.md`](./SETUP.md) — Meta 권한·해시태그 설정·허락 요청 흐름
- [`prompts/ugc-classify.md`](./prompts/ugc-classify.md) — sentiment·usability·pull_quote·formats 단일 호출
- [`workflows/ugc-collector.json`](./workflows/ugc-collector.json) — n8n 임포트용

## 폴더 구조

```
08-ugc-collector/
├── README.md
├── PRD.md
├── prompts/
├── workflows/
├── scripts/
└── docs/
```

## 의존성

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md) + [UGCLibrary DB](../../docs/notion-schemas/ugc-library.md), Meta IG 권한, 브랜드 멘션이 발생할 만한 인지도
- **활용**: 06(광고 진화)이 광고 시드, 10(LP A/B v2)이 LP 후기 섹션
- **부정 UGC**: 자동 차단 (sentiment=negative는 저장 안 함). 03 통합 인박스에서 별도 처리
