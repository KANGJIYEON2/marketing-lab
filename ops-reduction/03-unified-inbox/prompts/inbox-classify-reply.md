# inbox-classify-reply

들어오는 메시지 1건을 받아 **분류(intent·urgency·sentiment) + 답변 초안**을 한 번의 LLM 호출로 생성.

## 왜 한 번에?

분류와 답변 작성을 분리하면 LLM 호출 2회, 토큰 비용 2배, 일관성 ↓. 한 호출 안에 둘 다 시키면 LLM이 분류 결과를 답변 톤에 자연스럽게 반영함.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.4 (분류 정확성 + 답변 자연스러움 균형)
- max tokens: 800

## 시스템 메시지

```
You classify and draft replies for incoming customer messages across channels.

Brand voice: {BRAND_VOICE}

Return STRICT JSON:
{
  "intent": "question | complaint | praise | spam | sales_lead | other",
  "urgency": "high | medium | low",
  "sentiment": "positive | neutral | negative",
  "confidence": 0.0-1.0,
  "draft_reply": "<reply in the same language as the message; match channel tone>"
}

Channel tone notes:
- instagram_comment / instagram_dm: short, casual, 1-3 sentences max, emoji ok (max 1)
- gmail: more formal, full sentences, signature line ok
- youtube_comment / blog_comment: medium length, helpful tone

If intent is spam, set draft_reply to "" (empty).
If confidence < 0.6, escalate by appending "⚠️ 사람 확인 필요" at the end of draft_reply.
No prose, no markdown, no code fences.
```

`{BRAND_VOICE}`는 워크플로 `Config.brand_voice`.

## 사용자 메시지 (입력)

```
Channel: {channel}
Author: {author handle/email}
Received: {ISO timestamp}

Content:
{message body}
```

## 출력 예시

### 칭찬 (positive)
입력: `너무 잘 보고 있어요! 다음 영상도 기대할게요 ❤️`

```json
{
  "intent": "praise",
  "urgency": "low",
  "sentiment": "positive",
  "confidence": 0.95,
  "draft_reply": "감사합니다! 다음 영상도 열심히 준비할게요 🙌"
}
```

### 불만 (negative, high urgency)
입력: `결제했는데 영상이 안 열려요. 빨리 처리 부탁드립니다.`

```json
{
  "intent": "complaint",
  "urgency": "high",
  "sentiment": "negative",
  "confidence": 0.92,
  "draft_reply": "불편을 드려 죄송합니다. 결제 내역 확인해드릴게요. DM으로 주문 번호 보내주시면 5분 내로 조치하겠습니다."
}
```

### 영업 기회 (sales_lead)
입력: `회사에서 이 자동화 도입을 검토 중인데 미팅 가능하실까요?`

```json
{
  "intent": "sales_lead",
  "urgency": "high",
  "sentiment": "neutral",
  "confidence": 0.88,
  "draft_reply": "관심 가져주셔서 감사합니다. 회사 규모·도입 시기 간단히 알려주시면 적합한 미팅 시간 잡아드릴게요. 메일 주소 알려주시면 자료도 먼저 보내드립니다."
}
```

### 모호한 메시지 (낮은 confidence)
입력: `이거 진짜에요?`

```json
{
  "intent": "question",
  "urgency": "medium",
  "sentiment": "neutral",
  "confidence": 0.4,
  "draft_reply": "안녕하세요! '이거'가 어떤 부분 말씀하시는지 조금만 더 설명해주실 수 있을까요? ⚠️ 사람 확인 필요"
}
```

## 평가 기준 (첫 2주 매일 점검)

- [ ] `intent` 분류 정확률 > 80%
- [ ] `urgency` 분류 정확률 > 75% — 잘못 매기면 응대 누락 발생
- [ ] `confidence < 0.6` 비율 < 15% (이상이면 프롬프트 보강)
- [ ] `draft_reply`를 그대로 발송 가능한 비율 > 50% (수정 없이)
- [ ] 채널 톤이 가이드대로인가 (IG = 짧고 캐주얼, Gmail = 정중)

## 자주 발생하는 LLM 실수

1. **영업 톤 과잉** — `draft_reply`에 "지금 바로 구매하세요!" 같은 강매 문구
   → Brand voice에 "영업톤 자제" 명시
2. **너무 긴 IG 답변** — 3문장 넘어감
   → Channel tone notes에 "1-3 sentences max" 강조
3. **불만 사과 부족** — 문제 인정 없이 해결책만
   → `complaint`일 때 "Acknowledge first, then solve" 추가 가능
4. **이모지 폭주** — IG에서 5개씩
   → "emoji ok (max 1)" 명시

## 비용

- 입력: ~600 tokens (system + message)
- 출력: ~300 tokens
- 메시지 1건당 ≈ $0.005
- 일 100건 처리 = 월 $15
