# 10. 랜딩페이지 A/B 자동 실험

> 헤드라인·CTA·소셜프루프 변형 자동 생성 → 배포 → 위너 자동 채택

## 1. 페인포인트

- A/B 테스트 하고 싶지만 변형 만들고 코드 건드리는 게 귀찮음
- 한 번 만든 LP를 몇 달간 안 건드림 → 전환률 정체
- 통계적 유의성 판단을 매번 수동으로 하기 번거로움

## 2. KPI

| 지표 | 목표 |
|---|---|
| 월간 진행 실험 수 | 2개+ |
| 위너 결정까지 시간 | 자동 (유의성 도달 시) |
| LP 누적 CVR 개선 (분기 대비) | +20%+ |

## 3. Input / Output

**Input**
- 대상 LP URL
- 현재 컨트롤 버전 (헤드라인·서브헤드·CTA·소셜프루프 텍스트)
- 일평균 트래픽 (실험 기간 계산용)

**Output**
- LP 변형 5종 (헤드라인 위주) 또는 1:1 매칭 실험
- 배포: Webflow/Framer/Next.js LP에 variant 라우팅
- 위너 결정 + 자동 채택

## 4. n8n 워크플로

```
[수동 트리거: "이 LP A/B 실험 시작"]
        │
        ▼
[Claude API: 변형 생성]
  현재 카피 + 페르소나 + 콘텐츠 자산 →
  헤드라인 5개 변형 (각각 다른 후킹 앵글)
        │
        ▼
[Notion: Experiments DB 적재] status=ready
        │
        ▼
[사람 검토 → status=approved]
        │
        ▼
[LP 자동 배포]
  옵션 A: Webflow API (CMS field 업데이트)
  옵션 B: Framer API
  옵션 C: 자체 Next.js → variant 라우팅 (cookie)
        │
        ▼
[GA4: experiment_id custom dimension 발송]
        │
        ▼
[Cron: 매일 진행 체크]
  - GA4에서 variant별 CVR 조회
  - 통계 유의성 (Bayesian or Frequentist)
        │
        ▼
[유의성 도달 시]
  ├─▶ Slack 알림 "Variant 3이 +18% 위너"
  ├─▶ 위너로 컨트롤 자동 교체
  └─▶ Notion: status=completed, winner=3
```

## 5. 필요한 통합

- Claude API
- Notion API
- LP 플랫폼 API (Webflow/Framer/자체)
- GA4 API (서비스 계정)
- Slack
- 통계 라이브러리 (n8n function node에서 JS)

## 6. MVP 범위 (Week 7~8)

**포함**
- 헤드라인만 A/B (CTA/소셜프루프는 v2)
- 변형 2개 (A vs B, 5개는 v2)
- 위너 결정은 알림만, 교체는 수동

**제외 (v2)**
- 다중 요소 동시 테스트 (MVT)
- 자동 위너 채택
- 세그먼트별 분기 (모바일/데스크탑)

## 7. 의존성

- **선행**: 일평균 LP 트래픽 100+ (그 이하면 실험 무의미)
- **시너지**: 07(리드 시퀀스)이 전환 측정, 06(광고)이 트래픽 공급
- **연계**: 09 주간 리포트에 실험 현황 포함

## 8. 리스크

- 트래픽 부족 → 실험 시작 전 minimum sample size 계산해서 차단
- LP 플랫폼 제약 (Wix 등은 API 불완전) → 플랫폼 선택 단계에서 검토
- 변형 카피가 브랜드 톤 벗어남 → 브랜드 가이드 프롬프트 첨부

## 9. 프롬프트

`prompts/`:
- `lp-variant-headlines.md`
- `lp-variant-cta.md`
- `lp-variant-social-proof.md`
