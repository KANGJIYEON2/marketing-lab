# competitor-change-analyze

경쟁사 변경 1건(블로그 글 / 가격 페이지 diff / 신규 광고)을 받아 **요약 + 영향도 + 우리 액션**까지 단일 호출.

워크플로의 `Claude: Analyze Change` 노드에서 사용.

## 왜 단일 호출?

분석·점수·액션을 3번 호출하면 토큰 비용 3배 + 컨텍스트 단절. 한 번에 시키면 LLM이 자기 점수에 맞는 액션을 일관되게 생성.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.3 (분석은 결정적이어야)
- max tokens: 600

## 시스템 메시지

```
You analyze competitor activity from one of: blog post, pricing page change, or new ad creative.

Our context: {OUR_CONTEXT}

Return STRICT JSON:
{
  "summary": "<2-sentence summary of what changed/launched>",
  "category": "pricing | positioning | feature | content | launch | other",
  "impact_score": 0-10,
  "our_action": "<one specific action we should consider, or 'none — informational only'>",
  "reasoning": "<one sentence why impact_score>"
}

Impact scale:
- 0-3: informational, no action needed
- 4-6: worth tracking, may inform next quarter
- 7-8: should respond this week
- 9-10: urgent — competitor moved on our positioning

No prose, no markdown, no code fences.
```

`{OUR_CONTEXT}`은 `Config.our_context` (예: "1인 창업자용 마케팅 자동화 SaaS. 가격 $29~99/월. 차별점: n8n 기반 셀프호스팅 옵션.")

## 사용자 메시지 (입력)

```
Competitor: {name}
Source: blog | pricing | meta_ads
First snapshot: true | false
URL: {source_url}

Current content:
{raw_payload — 블로그 제목 또는 가격 페이지 텍스트 4000자 또는 광고 카피}

Previous snapshot (for diff):
{prev_text — pricing diff일 때만, 1500자}
```

## 출력 예시

### Case 1 — 경쟁사 가격 인하 (high impact)
```json
{
  "summary": "Competitor A가 Starter 플랜을 $29 → $19로 인하하고 무제한 워크플로 추가. 직접 가격대 경쟁.",
  "category": "pricing",
  "impact_score": 9,
  "our_action": "이번 주 안에 우리 가격 검토. 무제한 워크플로 매칭 vs 가치 차별화(셀프호스팅·고급 노드) 강조 중 결정.",
  "reasoning": "우리와 같은 가격대에서 한 단계 아래로 내려와 직접 경쟁 시작."
}
```

### Case 2 — 경쟁사 블로그 글 (informational)
```json
{
  "summary": "Competitor B가 'n8n vs Make 비교' 글 발행. 자사 제품을 둘 다 대체할 수 있는 옵션으로 포지셔닝.",
  "category": "content",
  "impact_score": 4,
  "our_action": "유사 비교 글 ('우리 vs n8n vs Make') 작성 검토. 우리 차별점(셀프호스팅 + AI 통합) 명시.",
  "reasoning": "직접 도발은 없으나 SEO 키워드 선점 시도. 1~2주 내 응답이 좋음."
}
```

### Case 3 — 신규 광고 (medium impact)
```json
{
  "summary": "Competitor A의 신규 Meta 광고: '5분 만에 마케팅 자동화' 후킹 + 무료 체험 14일.",
  "category": "positioning",
  "impact_score": 6,
  "our_action": "광고 카피의 '5분' 약속이 효과적인지 모니터링. 다음 우리 광고 변형에 '셋업 시간' 앵글 테스트.",
  "reasoning": "포지셔닝이 우리와 직접 충돌하지 않지만 같은 후킹 전략 모방 가능."
}
```

### Case 4 — 첫 스냅샷 (no prior data)
```json
{
  "summary": "Competitor C 가격 페이지 첫 수집. 3개 플랜 — Free, Pro $49, Business $149. 연간 결제 시 20% 할인.",
  "category": "pricing",
  "impact_score": 0,
  "our_action": "none — informational only",
  "reasoning": "비교 기준점만 확보. 변화 감지 시 다시 분석."
}
```

## 평가 기준

- [ ] `impact_score`가 일관성 있는가 (같은 점수의 변경들이 비슷한 무게감인가)
- [ ] `our_action`이 구체적인가 ("검토하라" 정도가 아니라 "이번 주 안에 X 결정")
- [ ] `summary`가 영문 원문도 한국어로 자연스럽게 정리하는가
- [ ] `category` 분류가 정확한가

## 흔한 실수

1. **과도한 impact_score** — 모든 변경을 6+ 점으로. 0-3 범위가 정상적으로 분포해야 함
2. **모호한 our_action** — "트렌드 모니터링", "동향 파악" 같은 액션 아닌 액션
3. **summary 영어 출력** — `Our context`가 한국어인데 summary는 영어. 시스템 메시지에 "Output summary in the language of OUR_CONTEXT" 추가 검토

## 비용

- 입력: ~1.5k tokens (system + payload)
- 출력: ~200 tokens
- 변경 1건당 ≈ $0.008
- 경쟁사 3개 × 일 평균 변경 2건 = 월 ~$1.5
