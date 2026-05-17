# lp-variant-headlines

LP 헤드라인 변형 N개를 다른 후킹 각도로 생성. 단순 어구 변경이 아닌 **value proposition framing 변화**.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.7 (다양성 우선)
- max tokens: 1500

## 시스템 메시지

```
You generate landing page headline variants for an A/B test.

Requirements per variant:
- Each variant uses a DIFFERENT hook angle (specific number / outcome / emotion / pain / contrast)
- Length: <= 60 chars
- Subhead suggestion (optional, <= 100 chars)
- Don't just rephrase — change the underlying value proposition framing
- Match the language of the control headline
- Keep it tonally consistent with the LP body content (provided as lp_text)

Return STRICT JSON:
{
  "variants": [
    {
      "variant_id": "A" | "B" | "C" | "D",
      "headline": "<60 chars>",
      "subhead": "<optional, 100 chars>",
      "hook_angle": "<which angle this variant uses>",
      "why": "<one sentence — why this might beat control for this persona>"
    }
  ]
}

Generate {N} variants labeled A, B, C... (do not include control).

No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
Target persona: {persona}
Hypothesis: {hypothesis or empty}

Control headline (current):
"{control}"

Control subhead (if any):
"{subhead}"

LP body context (truncated):
---
{first 4000 chars of LP text}
---
```

## 후킹 각도 5종 — 변형마다 다르게

LLM이 변형마다 다른 각도를 시도하도록 시스템 메시지가 강제. 5가지 표준 각도:

| 각도 | 설명 | 예시 (한국어) |
|---|---|---|
| **specific number** | 구체적 숫자·시간 | "5분 만에 셋업하는 마케팅 자동화" |
| **outcome** | 결과·약속 강조 | "월 15시간 회수하는 자동화 시스템" |
| **emotion** | 감정·정체성 | "혼자서도 5명 팀처럼 일하는 법" |
| **pain** | 페인 직격 | "마케팅 운영에 매일 2시간 빼앗기시나요?" |
| **contrast** | 비교·반전 | "n8n으로 Zapier 비용 1/10에 끝내는 법" |

추가 가능: question, social proof, authority, urgency, curiosity.

## 출력 예시

### Case 1 — 1인 창업자 LP

입력:
- control_headline: "마케팅 자동화 솔루션"
- target_persona: "1인 창업자"
- hypothesis: "구체적 시간 약속이 추상적 가치보다 전환 높을 것"

```json
{
  "variants": [
    {
      "variant_id": "A",
      "headline": "월 15시간을 회수해드립니다",
      "subhead": "1인 창업자 87명이 검증한 마케팅 자동화 시스템",
      "hook_angle": "specific number + social proof",
      "why": "1인 창업자는 시간이 가장 비싼 자원. 구체적 시간 약속이 추상적 '솔루션'보다 즉시 가치 전달."
    },
    {
      "variant_id": "B",
      "headline": "혼자서도 5명 팀처럼 일하세요",
      "subhead": "마케팅 콘텐츠·광고·CRM 자동화 18종 템플릿",
      "hook_angle": "emotion + identity",
      "why": "1인 창업자의 정체성 (혼자지만 효율적)을 자극. 자랑할 수 있는 결과를 약속."
    }
  ]
}
```

### Case 2 — 영문 SaaS LP

입력:
- control_headline: "Marketing automation for solo founders"
- target_persona: "solo SaaS founder"

```json
{
  "variants": [
    {
      "variant_id": "A",
      "headline": "Ship 5 campaigns in the time it took to ship 1",
      "subhead": "n8n-based automation, AI copy, 18 ready-to-go workflows",
      "hook_angle": "contrast — output multiplier",
      "why": "Founders measure progress by shipping cadence. Concrete multiplier beats vague 'automation' framing."
    },
    {
      "variant_id": "B",
      "headline": "Stop being your own marketing team",
      "subhead": "Reclaim 15h/week with self-hosted AI workflows",
      "hook_angle": "pain + outcome",
      "why": "Pain ('your own marketing team') is universal for solo founders. Outcome (15h/week) is measurable."
    }
  ]
}
```

## 평가 기준

- [ ] N개 변형이 서로 **다른 hook_angle** 사용 (모두 specific number면 실패)
- [ ] 각 헤드라인 ≤ 60자 (모바일 1줄)
- [ ] 서브헤드는 헤드라인을 보완 (반복 X)
- [ ] `why`가 짧고 명확 (이게 좋아야 사람이 검수 시 선택 가능)
- [ ] Control과 톤이 너무 다르지 않음 (브랜드 일관성)

## 흔한 LLM 실수

1. **단순 rephrase** — control을 그대로 다른 동사로. 가치 framing이 바뀌어야 의미 있는 실험
2. **모든 변형이 specific number** — 같은 각도 반복. 시스템 메시지에 "DIFFERENT hook angle" 강조
3. **너무 길어짐** — 80자+로 모바일에서 잘림
4. **subhead가 헤드라인 반복** — 헤드라인 = 약속, subhead = 증거/구체 보강
5. **과장 표현** — "100% 보장", "최고의" 등. 06 광고 가드레일 동일 적용

## 한국어 vs 영어 차이

- 한국어 60자 ≈ 영어 80-100 chars (글자수 차이)
- 시스템에 "Match the language"로 자동 조정
- 영어는 단순 명사 후킹이 잘 됨, 한국어는 동사·존재형이 자연스러움

## 변형 개수 권장

- **MVP는 2개**: Control vs Variant A (단순)
- **N=3~4**: 통계적 검증 충분, 검수 부담 적당
- **N≥5**: 트래픽이 충분히 많아야 (월 50k+ 세션). 그렇지 않으면 sample size 부족

## 비용

- 입력: ~5k tokens (lp_text + system)
- 출력: ~600 tokens
- 실험 1회당 ≈ $0.03
