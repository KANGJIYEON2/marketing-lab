# lead-score-classify

폼 제출 + Apollo enrich 결과를 받아 **스코어 + 페르소나 + 핫시그널**을 단일 호출로.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.2 (스코어링은 일관성 최우선)
- max tokens: 300

## 시스템 메시지

```
You score and classify inbound leads. Return STRICT JSON:
{
  "score": 0-100,
  "persona": "founder | marketer | dev | other",
  "hot_signal": true|false,
  "reasoning": "<one sentence>"
}

Score framework (start at 50, adjust):
+20 if persona=founder OR marketer
+15 if company_size >= 10
+15 if industry suggests fit (SaaS, agency, ecommerce)
+10 if message indicates clear use case or urgency
+10 if utm_source = paid (already invested attention)
-15 if free email domain (gmail/naver/yahoo)
-10 if message empty AND no enrichment data

hot_signal=true if score >= 70 OR message contains 'urgent'/'now'/'미팅'/'demo'.

No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
Email: {email}
Name: {name}
Company: {company or 'unknown'}
Role: {role or 'unknown'}
Industry: {industry or 'unknown'}
Company size: {employees or 'unknown'}
UTM source: {utm or 'direct'}
Message: {form message or '(none)'}
```

## 출력 예시

### High-value founder (hot)
```json
{
  "score": 85,
  "persona": "founder",
  "hot_signal": true,
  "reasoning": "Founder of 30-person SaaS, asked about demo this week — clear high-intent."
}
```

### Marketer at fit company
```json
{
  "score": 70,
  "persona": "marketer",
  "hot_signal": true,
  "reasoning": "Growth marketer at e-commerce, business email, free trial signup."
}
```

### Curious dev (low intent)
```json
{
  "score": 40,
  "persona": "dev",
  "hot_signal": false,
  "reasoning": "Indie dev with gmail, no message, just signed up to look around."
}
```

### Junk
```json
{
  "score": 10,
  "persona": "other",
  "hot_signal": false,
  "reasoning": "Free email, no message, no enrichment data — likely list scrape."
}
```

## 평가 기준 (첫 2주 매일 점검)

- [ ] `score` 분포가 0-100 전 구간에 퍼지는가 (모두 50 근처면 LLM이 보수적)
- [ ] hot_signal=true 비율 < 25% (이상이면 임계값 너무 낮음)
- [ ] hot 리드의 실제 미팅 전환율 > 30% (이하면 분류 정확도 ↓)
- [ ] `reasoning`이 한 문장으로 명확한가

## 페르소나 분류 힌트

| 신호 | persona |
|---|---|
| role에 "founder", "CEO", "owner" | founder |
| role에 "marketing", "growth", "content" | marketer |
| role에 "engineer", "developer", "CTO" | dev |
| 매칭 안 됨 | other |

free email + empty role + empty company는 보통 `other`.

## v2 강화 검토

1. **회사 도메인 기반 점수 추가** — 알려진 ICP 도메인 리스트와 매칭
2. **메시지 의도 분류** — `intent: trial | demo | partnership | support | spam`
3. **시점 페르소나** — 같은 사람이 1회 → 3회째 방문할 때 점수 가산

## 비용

- 입력: ~400 tokens
- 출력: ~120 tokens
- 리드 1건당 ≈ $0.003
- 월 1000 리드 = $3
