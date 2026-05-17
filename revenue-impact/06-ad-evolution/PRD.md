# 06. 광고 카피 진화 엔진

> 주간 광고 성과 분석 → 잘 된 패턴 추출 → 신규 변형 자동 생성·업로드

## 1. 페인포인트

- 광고 피로도(ad fatigue)로 ROAS가 매주 떨어짐
- 신규 크리에이티브 만들 시간 없음 → 같은 광고로 계속 돌림 → CPA 상승
- 어떤 후킹·CTA가 잘 되는지 데이터로 모르면 변형도 감으로 만듦

## 2. KPI

| 지표 | 목표 |
|---|---|
| 주간 신규 광고 변형 | 10개+ |
| 평균 ROAS (전월 대비) | +15% |
| 광고세트당 크리에이티브 수 | 3+ (피로도 분산) |

## 3. Input / Output

**Input**
- Meta Marketing API: 직전 7일 광고 성과 (ad-level)
- Google Ads API: 동일
- 콘텐츠 자산: 02 산출물 + 08 UGC + 블로그 글

**Output**
- 광고 변형 JSON (헤드라인·본문·CTA × 10개)
- Meta/Google에 자동 업로드 (Draft 상태 → 사람 검토 → 활성)

## 4. n8n 워크플로

```
[Cron: 매주 일요일 22시]
        │
        ▼
[Meta + Google Ads API: 직전 7일 성과]
  - ROAS, CTR, CPA, frequency per ad
        │
        ▼
[Top 20% 추출] ROAS 기준
        │
        ▼
[Claude API: 패턴 추출]
  "이 광고들의 공통 후킹 패턴, CTA 스타일,
   사용된 감정 트리거를 추출하라"
        │
        ▼
[Notion: 콘텐츠 자산 fetch]
  - 최근 발행 블로그 5편
  - UGC 라이브러리 상위 10개
        │
        ▼
[Claude API: 변형 생성]
  "추출한 패턴 + 콘텐츠 자산으로 신규 광고 카피 10개 생성.
   각각 헤드라인·본문(125자)·CTA."
        │
        ▼
[Notion: AdVariants 적재] status=draft
        │
        ▼
[병렬 이미지 생성]
  ├─▶ Midjourney API (또는 SD)
  └─▶ Bannerbear (텍스트 오버레이)
        │
        ▼
[Slack 알림] "10개 광고 변형 준비 — 검토: [Notion]"
        │
        ▼
[사람 검수 → approved]
        │
        ▼
[Meta/Google Ads API: 광고세트에 Draft 광고 생성]
```

## 5. 필요한 통합

- Meta Marketing API
- Google Ads API
- Claude API
- Notion API
- Midjourney API (또는 Stable Diffusion)
- Bannerbear (텍스트 오버레이 자동)
- Slack

## 6. MVP 범위 (Week 5~6)

**포함**
- Meta만 (Google은 v2)
- 카피만 (이미지 생성 v2)
- 업로드는 수동 (Slack 카피 메시지)

**제외 (v2)**
- 자동 이미지 생성
- 자동 업로드
- A/B 통계 자동 분석 (10번과 연계)

## 7. 의존성

- **선행**: 광고 운영 중 (월 광고비 $500+ 권장)
- **시너지**: 02 콘텐츠, 08 UGC → 카피 시드
- **연계**: 09 주간 리포트가 광고 성과 트래킹

## 8. 리스크

- Meta 광고 정책 위반 자동 체크 어려움 → 사람 검수 단계 유지
- LLM이 만든 카피의 클레임 표현(과대광고) → 프롬프트에 가드레일 + 금칙어 필터
- 변형 너무 많이 만들면 학습 분산 → 광고세트당 3~5개로 제한

## 9. 프롬프트

`prompts/`:
- `ad-pattern-extract.md`
- `ad-variant-generate.md`
- `ad-guardrails.md` (금칙어/과대표현 체크)
