# lp-variant-cta (v2)

⚠️ **MVP에 포함되지 않음.** MVP는 헤드라인만 A/B. CTA 변형은 v2에서 추가.

## 왜 v2로 분리하나

A/B 테스트의 핵심 원칙: **한 번에 한 변수만**. 헤드라인 + CTA 동시 변경하면 어느 변수가 효과를 냈는지 알 수 없음. 따라서:

1. 먼저 헤드라인 실험 → 위너 결정
2. 그 위너 위에 CTA 실험
3. 그 위에 social proof 실험
4. ... 순차적 누적

## v2 구현 방식

`workflows/generate-variants.json`의 webhook payload에 `variable: "cta"` 필드 추가:

```json
{
  "experiment_id": "exp_2026-05-24_cta",
  "lp_url": "https://yourdomain.com/pricing",
  "variable": "cta",
  "control_cta": "지금 시작하기",
  "variants_count": 3
}
```

워크플로가 `variable` 값에 따라 시스템 프롬프트 분기 (현재는 headline만).

## 예정된 시스템 메시지 (초안)

```
You generate CTA button text variants for an A/B test.

Requirements per variant:
- 2-5 words
- Action verb start
- Different commitment levels:
  * low: Learn More, See How, Read More
  * medium: Try Free, Start Trial, Get Started
  * high: Buy Now, Sign Up, Subscribe
- One variant per commitment level

Return STRICT JSON:
{
  "variants": [
    {
      "variant_id": "A" | "B" | "C",
      "cta": "<2-5 words>",
      "commitment_level": "low | medium | high",
      "why": "<one sentence>"
    }
  ]
}
```

## CTA 후킹 각도 5종

| 각도 | 예시 |
|---|---|
| **action verb** | "Try Free", "시작하기" |
| **outcome promise** | "Save 15 hours", "15시간 절약하기" |
| **risk reversal** | "Cancel Anytime", "언제든 해지" |
| **low friction** | "See Demo (no signup)", "체험 보기" |
| **scarcity** | "Get Early Access", "베타 신청" |

## 일반적 베스트 프랙티스

- **Soft ask > Hard ask** (대부분의 경우)
- **Verb-first** ("Get a quote" > "Quote request")
- **Specific > Vague** ("Start your 14-day trial" > "Start now")
- **First person 검토** ("Start my trial" — 일부 케이스에서 +20%)

## 구현 시점 결정 기준

- 헤드라인 실험 1~2회 완료 후
- 트래픽이 충분히 누적된 후 (월 10k+ 세션)
- 컨버전이 측정 가능한 정도 (월 100+ 컨버전)

당분간 MVP의 headline 실험만 운영하다 점차 추가.
