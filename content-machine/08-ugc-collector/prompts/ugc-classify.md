# ugc-classify

수집된 UGC 1건을 받아 **sentiment + usability + pull_quote + usage_formats**까지 단일 호출.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.3
- max tokens: 600

## 시스템 메시지

```
You analyze a user-generated post mentioning a brand to determine if it's usable as marketing asset.

Return STRICT JSON:
{
  "sentiment": "positive | neutral | negative",
  "usability_score": 0-10,
  "pull_quote": "<the most quotable single sentence, or empty if none>",
  "usage_formats": ["carousel", "ad_creative", "lp_testimonial", "repost", "case_study"],
  "reasoning": "<one sentence on why this score>"
}

Usability score guide:
- 0-3: low quality, generic, no specifics, vague
- 4-6: ok content but missing key elements (specific result, emotion, story)
- 7-8: clear positive experience, quotable, has specifics
- 9-10: exceptional — specific numbers, emotional story, transformative

usage_formats array: include ONLY the formats that fit this content. Be selective — most posts fit 1-2 formats max.
- carousel: visual-first post with photo + commentary
- ad_creative: short, punchy, quotable text
- lp_testimonial: full sentences about outcome/benefit
- repost: photo-heavy, brand visible
- case_study: detailed story with context

Negative sentiment = usage_formats empty, usability_score 0. Negative content needs ops routing, not asset use.

No prose, no markdown, no code fences. Match content language.
```

## 사용자 메시지 (입력)

```
Channel: {channel}
Author: {author} ({@handle})
Media URL: {url or '(none)'}
Posted: {timestamp}

Content:
{post caption/text}
```

## 출력 예시

### Case 1 — 구체적 후기 (high usability)

입력: 캡션 "n8n으로 매주 10시간씩 줄였어요! 광고 카피 자동 생성이 진짜 쩔어요 ㅋㅋ"

```json
{
  "sentiment": "positive",
  "usability_score": 9,
  "pull_quote": "n8n으로 매주 10시간씩 줄였어요",
  "usage_formats": ["ad_creative", "lp_testimonial"],
  "reasoning": "구체적 숫자(10시간) + 명확한 가치(시간 절약) + 실제 사용 사례. 광고 카피와 LP 후기 모두에 직접 활용 가능."
}
```

### Case 2 — 일반적 칭찬 (medium)

입력: "오늘도 잘 쓰고 있어요 감사합니다 :)"

```json
{
  "sentiment": "positive",
  "usability_score": 4,
  "pull_quote": "",
  "usage_formats": ["repost"],
  "reasoning": "긍정적이지만 구체성 부족. 인용보다는 스토리에 리포스트 정도가 적합."
}
```

### Case 3 — 비주얼 중심 (carousel fit)

입력: 캡션 "오늘 자동화 완성! @brand 진짜 직관적이네요" + media_url 존재

```json
{
  "sentiment": "positive",
  "usability_score": 7,
  "pull_quote": "@brand 진짜 직관적이네요",
  "usage_formats": ["carousel", "repost"],
  "reasoning": "사진 있는 짧은 호평. 캐러셀 슬라이드나 IG 스토리 리포스트에 적합. 글 자체는 짧아서 광고 카피보다는 비주얼 활용."
}
```

### Case 4 — 부정 (ops route)

입력: "결제했는데 작동 안 됨. 환불 부탁드립니다."

```json
{
  "sentiment": "negative",
  "usability_score": 0,
  "pull_quote": "",
  "usage_formats": [],
  "reasoning": "환불 요청 — 자산화 절대 금지. 03 통합 인박스 또는 운영팀으로 라우팅 필요."
}
```

### Case 5 — 영문 후기

입력: "Just signed up for {brand} and it cut my Monday meeting prep from 2h to 15min. Wild."

```json
{
  "sentiment": "positive",
  "usability_score": 8,
  "pull_quote": "cut my Monday meeting prep from 2h to 15min",
  "usage_formats": ["ad_creative", "lp_testimonial", "case_study"],
  "reasoning": "Specific time savings + clear ROI framing. Excellent for English-language ads and LP testimonials."
}
```

## 평가 기준 (첫 2주)

- [ ] sentiment 분류 정확률 > 90%
- [ ] usability_score 7+로 분류된 것 중 사람이 봤을 때 실제 자산 가치 있는 비율 > 70%
- [ ] pull_quote가 그대로 광고/LP에 쓸 수 있는 형태 (광고 정책 위반 X)
- [ ] usage_formats가 1-2개로 한정 (5개 다 매칭하는 경우 → 너무 관대)

## 흔한 LLM 실수

1. **모든 긍정 = 9점** — 캐주얼 칭찬과 진짜 후기 구분 안 됨. 위 가이드대로 0-10 분포 강제
2. **usage_formats 5개 다 포함** — 보수적으로. 1-2개가 정상
3. **부정 콘텐츠에 usability 점수** — sentiment=negative면 무조건 0점, formats 비움
4. **pull_quote에 이모지** — 이모지 제거하고 깔끔한 문장만

## 허락 받기 가이드 (사람 운영용)

자동화는 수집·분류까지. 허락 요청 메시지 표준:

> 안녕하세요! @{브랜드명}입니다. {원본 게시물 링크} 후기 정말 감사합니다. 저희 마케팅 채널/광고에 인용해도 괜찮으실까요? 출처(@핸들)는 명시하겠습니다.

답변 받으면 Notion에서 `permission_status` 수동 업데이트.

## 비용

- 입력: ~600 tokens
- 출력: ~250 tokens
- 1건당 ≈ $0.005
- 일 30건 처리 = 월 $4.5
