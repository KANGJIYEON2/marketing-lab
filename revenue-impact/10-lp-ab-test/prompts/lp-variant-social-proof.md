# lp-variant-social-proof (v2)

⚠️ **MVP에 포함되지 않음.** Social proof 변형도 v2.

## 소셜 프루프 종류

1. **고객 수** — "1,200 founders가 사용 중"
2. **인용 (testimonial)** — 실제 후기 [08 UGCLibrary](../../content-machine/08-ugc-collector/) 활용
3. **로고 월** — 유명 고객사 로고
4. **숫자 통계** — "월 평균 15시간 절약", "ROAS 3.2x"
5. **미디어 노출** — "TechCrunch, Product Hunt 1위"
6. **인증·수상** — ISO, SOC2, Y Combinator 등

## v2 구현 방식

`workflows/generate-variants.json`에 `variable: "social_proof"` 추가:

```json
{
  "experiment_id": "exp_2026-06-07_social",
  "lp_url": "https://yourdomain.com/pricing",
  "variable": "social_proof",
  "control_proof": "300+ founders trust us",
  "available_assets": {
    "customer_count": 1200,
    "testimonials": ["pull_quote 1", "pull_quote 2"],
    "logos": ["Stripe", "Vercel"],
    "stats": ["월 15시간 절약", "ROAS 3.2x"],
    "media": ["Product Hunt #1"]
  },
  "variants_count": 3
}
```

## UGC 연동

08의 `UGCLibrary` DB에서 `permission_status=approved AND usability_score>=8` 행 자동 fetch:
- `pull_quote` 필드 → testimonial 변형
- `author_handle` → 출처 표기

워크플로에 추가될 노드:
```
[Fetch UGCLibrary approved]
   ↓
[Pick top 3 quotes by usability_score]
   ↓
[Build social_proof variants]
```

## 예정 시스템 메시지 (초안)

```
You generate social proof variants for an A/B test.

Each variant uses a DIFFERENT proof type from: customer_count, testimonial, logo_wall, stat, media, certification.

Return STRICT JSON:
{
  "variants": [
    {
      "variant_id": "A" | "B" | "C",
      "proof_type": "customer_count | testimonial | logo_wall | stat | media | certification",
      "content": "<the proof element copy>",
      "render_hint": "<how to display: text-only | with-photo | grid>",
      "why": "<one sentence>"
    }
  ]
}
```

## 평가 기준 (v2 구현 시)

- 각 변형이 다른 proof_type
- 거짓·과장 없음 (실제 보유한 자산 기반만)
- 출처가 명확함 (testimonial의 경우 author)

## 우선순위

10 LP A/B의 v2 확장 순서:
1. **CTA 실험** (가장 간단, 효과 큼)
2. **Social proof 실험** (UGC와 연계)
3. **Hero image 실험** (이미지 자동 생성 + 측정, 가장 복잡)
4. **Multi-variate testing** (여러 변수 동시, 트래픽 충분 시)

## 한국 시장 주의

- 영문권에서 잘 통하는 로고 월이 한국에서는 비교적 약함
- 한국에서는 **숫자 통계 + 실명 후기**가 강력
- "인스타 1만 팔로워" 같은 한국식 social proof도 효과

## 구현 시점

MVP 운영 후 2~3개월. UGCLibrary가 어느 정도 쌓이고, 헤드라인 실험 패턴이 안정된 후.
