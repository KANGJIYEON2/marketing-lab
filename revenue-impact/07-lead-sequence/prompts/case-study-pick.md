# case-study-pick (v2)

⚠️ **MVP에 포함되지 않음.** Day 0 단일 메일만 보내는 MVP에서는 사용하지 않는다. Day 2 케이스 스터디 메일을 만들 때 사용할 프롬프트 명세.

## 목적

Day 2 메일에서 리드의 페르소나·산업·메시지 의도에 가장 맞는 케이스 스터디 1개를 골라 본문 작성.

## v2 워크플로 구조

```
[Cron: 매일 09:00]
   ↓
[Notion: Leads where sequence_status = day0_sent AND day0_sent_at >= 2일 전]
   ↓
[For each lead]
   ↓
[Claude: case-study-pick] — 페르소나·메시지 기반 적합한 케이스 1개 선택
   ↓
[Claude: write Day 2 email] — 선택된 케이스 본문 + 리드 컨텍스트
   ↓
[Resend: Send]
   ↓
[Notion: sequence_status = day2_sent]
```

## 케이스 스터디 풀 (예시)

본인 사이트의 실제 케이스 스터디로 교체. 권장: 3~5개 보유.

```yaml
case_studies:
  - id: "saas-onboarding"
    title: "SaaS 회사의 온보딩 메일 시퀀스 자동화"
    fit_personas: [founder, marketer]
    fit_industries: [saas]
    summary: "ARR $500k SaaS가 7일 시퀀스를 1주만에 셋업, 전환률 18% → 31%."
    url: "https://yourdomain.com/case/saas-onboarding"

  - id: "ecommerce-cart-abandon"
    title: "이커머스 장바구니 이탈 회수 자동화"
    fit_personas: [founder, marketer]
    fit_industries: [ecommerce, retail]
    summary: "월 매출 $30k 쇼피파이 스토어가 카트 이탈 회수 자동화로 매출 22% 증가."
    url: "https://yourdomain.com/case/ecommerce-cart"

  - id: "agency-reporting"
    title: "마케팅 에이전시의 클라이언트 리포팅 자동화"
    fit_personas: [marketer, other]
    fit_industries: [agency, marketing]
    summary: "8명 에이전시가 주간 클라이언트 리포트를 사람 0명으로 굴림."
    url: "https://yourdomain.com/case/agency-reporting"
```

## 시스템 메시지 (초안)

```
You select the most relevant case study for an inbound lead from a fixed pool.

Pool: {CASE_STUDY_LIST_JSON}

Lead info:
{lead context}

Selection rules:
1. Match persona first
2. Then industry
3. If no match → pick the most generally applicable case

Return STRICT JSON:
{
  "case_id": "<id from pool>",
  "why_match": "<one sentence>",
  "subject_hint": "<subject line idea>"
}
```

그 다음 별도 호출로 케이스 스터디 본문을 활용한 Day 2 메일 작성.

## 평가 기준 (v2 구현 시)

- [ ] 케이스 매칭률 — 사람이 봤을 때 적절한 비율 > 70%
- [ ] Day 2 메일 오픈률 > 35%
- [ ] Day 2 → 답장률 > 5%

## 구현 시점 결정 기준

MVP 운영 후:
- Day 0만으로도 답장률 > 8%면 시퀀스 불필요할 수도 (가성비 검토)
- Day 0 답장률 < 3%면 시퀀스로 보강 필요 → v2 구현 우선순위 ↑
- 트래픽이 충분해 시퀀스 효과 측정 가능할 때 (월 200 리드+)

본 프롬프트 파일은 그때 활용할 명세서로 남겨둠.
