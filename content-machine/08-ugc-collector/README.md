# 08 · UGC·리뷰 컬렉터

> 외부 리뷰·태그·멘션 자동 수집 → 캐러셀·광고·LP 소재화

**카테고리**: [content-machine](../README.md) · **로드맵**: Week 7 · **상태**: PRD only

## 진입점

- [`PRD.md`](./PRD.md) — 페인포인트·KPI·워크플로 다이어그램·MVP 범위
- `SETUP.md` — 구현 시 작성 예정
- `prompts/ugc-classify.md` — 구현 시 추가
- `workflows/ugc-collector.json` — n8n 임포트용 (구현 시 추가)

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

- **선행**: [Layer 0](../../docs/layer-0/SETUP.md), 브랜드 멘션이 발생할 만한 인지도
- **활용**: 수집 결과를 06(광고 진화)이 광고 시드로 사용
