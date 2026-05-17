# ad-variant-generate

추출된 패턴 + 콘텐츠 시드 → 새 광고 카피 10개.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.7 (variants는 다양해야 함)
- max tokens: 3000

## 시스템 메시지

```
You generate Meta ad copy variants based on winning patterns and content seeds.

Our product: {OUR_PRODUCT_PITCH}

Requirements per variant:
- headline: <= 40 chars (Meta primary text)
- body: 90-125 chars (Meta primary text body)
- cta: 2-3 words, action verb (Learn More, Try Free, Get Started, See How)
- pattern_used: which winning pattern it applies (from hook_patterns/cta_patterns input)

Return STRICT JSON:
{
  "variants": [
    {"headline": "...", "body": "...", "cta": "...", "pattern_used": "..."},
    ...
  ]
}

Generate {N} variants.

Guardrails (skip variants that violate these):
- No superlatives ("best", "#1", "the only")
- No income claims ("earn $X", "guaranteed results")
- No before/after weight or appearance
- No urgency lies ("last chance" when not true)
- No clickbait ("you won't believe", "this one trick")

Use content seeds as topic inspiration but write fresh ad copy (don't just paste blog titles).
Match language: if seeds are Korean, variants in Korean.

No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
Winning patterns:
Hooks: {pattern 1 | pattern 2 | ...}
CTAs: {pattern 1 | pattern 2 | ...}
Emotional triggers: {trigger 1 | trigger 2 | ...}
Avoid: {pattern 1 | pattern 2 | ...}

This week's insight: {summary}

Content seeds (recent published blog posts):
[{title, meta_desc, target_keyword, url}, ...]
```

## 출력 예시 (한국어 광고)

입력 패턴: "구체적 숫자 + 타겟 페르소나 + 낮은 약속 CTA"
입력 시드: ["1인 창업자를 위한 n8n 자동화 7가지", "마케팅 시간 주 15시간 줄이는 법"]

```json
{
  "variants": [
    {
      "headline": "1인 창업자, 15시간 회수했어요",
      "body": "주 15시간씩 가져가던 반복 마케팅 — n8n 7개 자동화로 15분으로 줄였습니다. 셀프호스팅 가능.",
      "cta": "보러가기",
      "pattern_used": "구체적 숫자 + 타겟 페르소나"
    },
    {
      "headline": "마케팅 자동화, 5분이면 됩니다",
      "body": "Notion·Slack·Meta 다 연결. AI가 카피 쓰고 광고 만들어요. 1인 사업자도 가능.",
      "cta": "시작하기",
      "pattern_used": "구체적 시간 + 즉시성"
    },
    {
      "headline": "Founder 분들이 가장 먼저 하는 자동화",
      "body": "고객 댓글 답변, 콘텐츠 리퍼포징, 리드 응대. 처음 5개 자동화 그대로 공유합니다.",
      "cta": "받아보기",
      "pattern_used": "타겟 페르소나 + 사회적 증명"
    }
    // ... 7개 더
  ]
}
```

## 가드레일 작동 예시

LLM이 다음과 같이 만들면 안 됨:

❌ `headline: "최고의 마케팅 자동화 #1"` — superlative
❌ `body: "월 매출 1억 보장!"` — income claim
❌ `body: "오늘이 마지막 기회!"` — urgency lie
❌ `headline: "이거 진짜 안 보면 후회"` — clickbait

LLM이 위반하면 해당 variant는 사람이 검수 단계에서 폐기. v2에서는 자동 필터링 추가.

## 평가 기준

- [ ] 10개 변형이 서로 다른 패턴 사용 (다양성)
- [ ] 모두 가드레일 통과
- [ ] headline 40자 / body 90-125자 길이 준수
- [ ] CTA가 2-3단어, 행동 동사
- [ ] 콘텐츠 시드의 토픽 활용하되 카피는 fresh

## 흔한 LLM 실수

1. **시드 제목 복붙** — 블로그 제목 그대로 광고 카피로. 광고는 새 카피여야 함
2. **모든 변형 비슷한 톤** — 한 패턴만 반복. temperature 0.7 + 시스템에 "diverse" 강조
3. **CTA 너무 강함** — "Buy Now!", "Sign Up Today!". 광고 효과 ↓. soft ask가 보통 잘 작동
4. **본문 너무 김** — Meta는 모바일에서 자름. 90-125자 엄수
5. **가드레일 위반** — superlative 사용. 시스템 메시지에 명시적 금지 목록 강조

## 검수 흐름

1. 일요일 밤 자동 생성 → Notion ContentDrafts에 `status=review`로 적재
2. 월요일 아침 Slack 알림 받음
3. 사람이 검수 — 10개 중 좋은 것 3~5개 선택
4. 선택된 변형 → 직접 Meta Ads Manager에 업로드 (MVP는 수동)
5. 2주 후 성과 측정 → 좋은 변형은 시드로 재사용 (자연스러운 진화 루프)

## v2 자동 업로드

PRD `MVP 범위 (제외)` 항목 — Meta Marketing API의 `/adcreatives` + `/ads` endpoint로 Draft 상태 광고 자동 생성. 검수는 Meta Ads Manager UI에서.

## 비용

- 입력: ~1.5k tokens (patterns + seeds + system)
- 출력: ~2.5k tokens (10개 변형)
- 주 1회 = 월 ≈ $0.30
