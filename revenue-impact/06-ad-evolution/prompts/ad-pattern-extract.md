# ad-pattern-extract

지난 주 광고 성과(Top 20% vs Bottom 20%) → 전이 가능한 패턴 추출.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.4 (분석은 결정적이지만 패턴 명명에 약간의 창의성)
- max tokens: 800

## 시스템 메시지

```
You analyze top-performing vs bottom-performing Meta ads to extract winning patterns.

Return STRICT JSON:
{
  "hook_patterns": ["<pattern type, e.g. 'specific number in first 5 words'>", "..."],
  "cta_patterns": ["<pattern type, e.g. 'soft ask — Learn → Read'>", "..."],
  "emotional_triggers": ["<e.g., FOMO, social proof, savings>", "..."],
  "avoid_patterns": ["<patterns common in losers, e.g. 'vague feature lists', 'industry jargon'>", "..."],
  "summary": "<1 sentence: what wins in this account this week>"
}

Focus on transferable patterns, not specific copy. Output language must match the ad copy language.
No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
Top 20% ads (ROAS-sorted, highest first):
[{name, roas, ctr, spend}, ...]

Bottom 20% ads:
[{name, roas, ctr, spend}, ...]

⚠️ ad_name만 보임 (Meta Insights API는 카피 본문 미제공). 이름·캠페인명에서 추론 가능한 패턴만 추출.
```

## 중요 한계 — Meta Insights API의 카피 본문 미제공

Meta Marketing API의 `/insights` endpoint는 `ad_creative` 본문을 직접 제공하지 않는다. 본문 조회는 별도 호출 (`/{ad_id}?fields=creative{...}`) 필요. MVP에서는 비용·복잡도 이유로 ad_name과 adset_name만 사용.

**한계의 영향**:
- LLM이 본문 패턴 (구체적 문장 구조)을 추출하지 못함
- ad_name이 "Hook A | Audience X | Variant 3" 같은 의미 있는 명명 규칙이면 추출 가능
- ad_name이 무의미한 ID면 추출 품질 낮음

**대응**:
1. **즉시**: ad_name을 의미 있게 짓는 규칙 도입 (`{hook}-{audience}-{variant}` 형식)
2. **v2**: ad_creative 본문 호출 추가 (Meta API 호출 N+1 증가, 비용 미세하지만 분석 품질 ↑)

## 출력 예시

### 데이터 풍부한 경우 (좋은 ad_name 규칙)

입력 ad_names:
- Top: "5min-setup-founder-v3", "ROAS-claim-saas-v1", "Sunday-savings-marketer-v2"
- Bottom: "feature-list-generic-v1", "industry-jargon-marketer-v4"

```json
{
  "hook_patterns": [
    "구체적 숫자 (5min, 7-day) 첫 단어로 등장",
    "결과 약속 (ROAS, savings) 첫 줄에 명시",
    "타겟 페르소나 명시 (founder/marketer)"
  ],
  "cta_patterns": [
    "구체적 시간 약속 (start in 5 min)",
    "낮은 약속 (Try, Learn, See — 'Buy/Sign up' 아님)"
  ],
  "emotional_triggers": [
    "즉시성 (5min, today)",
    "사회적 증명 (founder community)",
    "절약 (savings)"
  ],
  "avoid_patterns": [
    "기능 나열 (feature-list)",
    "산업 전문 용어 (industry-jargon)",
    "버전 v4까지 가도 변형 안 한 광고"
  ],
  "summary": "이 계정에서는 '구체적 시간 + 타겟 페르소나 명시'가 가장 잘 작동. 기능 나열은 즉시 폐기."
}
```

### 데이터 부족한 경우 (의미 없는 ad_name)

입력 ad_names:
- Top: "Ad 1", "Ad 17", "Ad 23"
- Bottom: "Ad 4", "Ad 9", "Ad 12"

```json
{
  "hook_patterns": [],
  "cta_patterns": [],
  "emotional_triggers": [],
  "avoid_patterns": [],
  "summary": "ad_name이 의미 없어 패턴 추출 불가. ad_name 명명 규칙을 'hook-audience-variant' 형식으로 도입 권장."
}
```

이 경우 변형 생성 단계에서는 generic best practices를 사용하게 됨. 한계.

## 평가 기준

- [ ] 각 카테고리 1~4개 패턴 (너무 많으면 노이즈)
- [ ] 패턴이 **transferable** — 다른 광고에도 적용 가능한 추상 수준
- [ ] avoid_patterns가 비어 있지 않은가 (실패 패턴 인식은 성공 패턴만큼 중요)
- [ ] summary가 한 줄로 인사이트 전달

## 비용

- 입력: ~1k tokens (top/bottom 메타 + system)
- 출력: ~400 tokens
- 주 1회 호출 = 월 ≈ $0.02
