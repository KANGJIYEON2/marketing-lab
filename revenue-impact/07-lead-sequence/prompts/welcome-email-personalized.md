# welcome-email-personalized

Day-0 환영 메일을 리드 정보 기반으로 개인화. 회사명·역할·메시지·hot 여부를 모두 반영.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.6 (자연스러움 우선, 일관성과 균형)
- max tokens: 1000

## 시스템 메시지

```
You write a Day-0 welcome email to an inbound lead.

Requirements:
- Length: 80-150 Korean characters body (≈400-600 chars in English)
- Tone: warm but professional, 1인칭 단수, no marketing fluff
- Reference 1 specific detail from their context (company/role/industry/message)
- Offer ONE concrete next step (a relevant resource link OR a 15-min call link)
- Do NOT include a generic "learn more about us" link
- Match language: if name/message looks Korean, write in Korean; else English
- Subject line MUST be under 60 chars and reference their context

Return STRICT JSON:
{
  "subject": "<email subject>",
  "body_html": "<HTML body, simple <p>/<a> only, no styling>",
  "body_text": "<plain text version>"
}

No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
Lead info:
Name: {name}
Email: {email}
Company: {company or 'unknown'}
Role: {role or 'unknown'}
Industry: {industry or 'unknown'}
Persona: {founder|marketer|dev|other}
Message from form: {message or '(no message)'}
Hot signal: {true|false}

Resource pool (pick ONE that matches their persona/message):
- founder: "Marketing automation playbook for solo founders" → https://yourdomain.com/playbook
- marketer: "5 n8n templates we use in production" → https://yourdomain.com/templates
- dev: "Self-host n8n with AI nodes in 10 min" → https://yourdomain.com/selfhost
- other: "How we built our marketing automation" → https://yourdomain.com/story

If hot_signal=true, replace resource with a 15-min call invite: https://cal.com/yourname/15min
```

본인 사이트의 실제 URL로 교체.

## 출력 예시

### Hot 리드, founder, 한국어

입력: name=김지영, role=대표, company=Acme(SaaS), message="이번 주 미팅 가능할까요?"

```json
{
  "subject": "지영님, 이번 주 15분 통화 — Acme 자동화 도입 건",
  "body_html": "<p>김지영 대표님 안녕하세요,</p><p>Acme에서 자동화 도입 검토 중이시라는 메시지 봤습니다. SaaS 초기 단계 founder분들 가장 자주 부딪히는 지점들 짧게 정리해서 들고 갈게요.</p><p>이번 주 가능한 시간 골라주시면 15분 통화 잡겠습니다:</p><p><a href=\"https://cal.com/yourname/15min\">cal.com/yourname/15min</a></p><p>감사합니다.</p><p>—<br>홍길동</p>",
  "body_text": "김지영 대표님 안녕하세요,\n\nAcme에서 자동화 도입 검토 중이시라는 메시지 봤습니다. SaaS 초기 단계 founder분들 가장 자주 부딪히는 지점들 짧게 정리해서 들고 갈게요.\n\n이번 주 가능한 시간 골라주시면 15분 통화 잡겠습니다:\nhttps://cal.com/yourname/15min\n\n감사합니다.\n\n—\n홍길동"
}
```

### Cold 리드, marketer, 한국어

입력: name=박철수, role=Growth Marketer, company=커머스 회사, message=null

```json
{
  "subject": "철수님, 커머스에서 실제로 쓰고 있는 n8n 템플릿 5개",
  "body_html": "<p>박철수님 안녕하세요,</p><p>이커머스에서 마케팅 자동화 굴리실 때 가장 자주 받는 질문이 \"어떤 워크플로부터 만들면 되나\"입니다. 저희 운영 중인 5개 템플릿 그대로 공유드려요.</p><p><a href=\"https://yourdomain.com/templates\">템플릿 5종 보기</a></p><p>도움 되셨으면 답장 주세요. 본 워크플로 하나 정도 같이 다뤄드릴 수 있습니다.</p><p>—<br>홍길동</p>",
  "body_text": "박철수님 안녕하세요,\n\n이커머스에서 마케팅 자동화 굴리실 때 가장 자주 받는 질문이 \"어떤 워크플로부터 만들면 되나\"입니다. 저희 운영 중인 5개 템플릿 그대로 공유드려요.\n\n템플릿 5종 보기: https://yourdomain.com/templates\n\n도움 되셨으면 답장 주세요. 본 워크플로 하나 정도 같이 다뤄드릴 수 있습니다.\n\n—\n홍길동"
}
```

## 평가 기준

- [ ] 첫 1줄에 받는 사람 이름이 자연스럽게 등장
- [ ] 본문 안에 회사/역할/메시지 중 적어도 1개 구체적 디테일이 인용됨
- [ ] subject가 60자 이내, 일반 환영 메일 같지 않음
- [ ] 한 메일에 CTA 1개 (링크 1~2개, 절대 3개 이상 X)
- [ ] 영업 톤 없음 ("지금 구매하세요!" 금지)
- [ ] HTML이 단순 (style/class 없음, p/a/br만)

## 흔한 LLM 실수

1. **이름 없이 시작** — "안녕하세요" 만. 이름 빠지면 개인화 효과 ↓
2. **회사명 거론 안 함** — Apollo enrich 결과를 활용 안 하면 매뉴얼 발송과 차이 없음
3. **CTA 3개 이상** — "자료 보기 / 문의하기 / 무료 체험" 다 넣음
4. **너무 긴 본문** — 200자 넘으면 모바일에서 잘림
5. **HTML 스타일 인라인** — `<p style="...">` 같은 인라인 스타일. 메일 클라이언트마다 깨질 수 있어 금지

## 도메인 인증 필수

이 프롬프트가 아무리 좋아도 메일이 스팸 폴더로 가면 무용지물. Resend 발신 도메인의 **SPF / DKIM / DMARC** 인증 필수. [SETUP.md](../SETUP.md) §3 참고.

## v2 강화

1. **3단계 시퀀스** — Day 2 (케이스 스터디), Day 5 (미팅 제안). 별도 워크플로 또는 스케줄러
2. **답장 자동 분류** — IMAP/Resend webhook으로 답장 감지 → 분류 → 라우팅
3. **A/B 변형** — subject 2개 생성해서 50/50 split

## 비용

- 입력: ~700 tokens
- 출력: ~500 tokens
- 메일 1건당 ≈ $0.012
- 월 500 리드 = $6
