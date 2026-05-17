# 07. 리드 캡처 → 개인화 환영 시퀀스

> 폼 제출 → 자동 enrich → 회사 상황에 맞춘 환영 메일 3단계

## 1. 페인포인트

- 일반 환영 메일 vs 개인화 메일은 전환률 3~5배 차이
- 수동 enrich + 작성은 리드 1명당 10~15분 → 확장 불가
- 응답 속도가 늦으면(>1시간) MQL 전환률 80% 떨어짐

## 2. KPI

| 지표 | 목표 |
|---|---|
| 첫 응답 시간 | < 5분 |
| 리드 → MQL 전환률 | 25%+ |
| 시퀀스 오픈률 | 50%+, 답장률 10%+ |

## 3. Input / Output

**Input**
- 폼 제출 webhook (Tally, Typeform, 또는 자체 LP)
- 최소 필드: 이메일, 이름 / 선택: 회사, 직책

**Output**
- 환영 메일 3단계 (Day 0, Day 2, Day 5)
- Notion `Leads` DB 적재
- Slack 핫리드 알림

## 4. n8n 워크플로

```
[Webhook: 폼 제출]
        │
        ▼
[Enrich]
  ├─▶ Apollo.io API (이메일 → 회사·직책·도메인)
  ├─▶ Clearbit (대안)
  └─▶ LinkedIn 검색 (Phantombuster, 옵션)
        │
        ▼
[Claude API: 리드 스코어링 + 페르소나 분류]
  - score 0-100
  - persona: founder / marketer / dev / other
  - hot_signal: yes/no
        │
        ▼
[Notion: Leads 적재]
        │
        ▼
[Hot signal이면] → Slack 즉시 알림
        │
        ▼
[Claude API: 개인화 메일 3종 생성]
  Day 0: 환영 + 가장 관련 있는 리소스 1개
  Day 2: 케이스 스터디 (페르소나 매칭)
  Day 5: 미팅 제안 (hot이면) / 다음 자료 (warm이면)
        │
        ▼
[Resend/Mailchimp: 시퀀스 큐 등록]
        │
        ▼
[답장 감지: IMAP 또는 webhook]
  → Slack 알림 + Notion status=replied
```

## 5. 필요한 통합

- 폼: Tally / Typeform webhook
- Enrich: Apollo.io 또는 Clearbit
- Claude API
- Notion API
- Resend (권장, 개발자 친화) 또는 Mailchimp
- Slack

## 6. MVP 범위 (Week 4)

**포함**
- 폼 → enrich → Day 0 환영 메일 (단일 메일)
- Notion 적재, Slack 알림
- 개인화는 회사명·직책 수준

**제외 (v2)**
- 3단계 시퀀스 전체 (Day 2, Day 5)
- 페르소나 분기
- 답장 자동 분류·라우팅

## 7. 의존성

- **독립 실행 가능**
- **시너지**: 10(LP A/B)이 폼 트래픽 공급, 09(리포트)에서 전환 트래킹

## 8. 리스크

- Enrich 비용: Apollo는 리드당 ~$0.05 → 월 리드 수 추정 후 플랜 결정
- 메일 도달률: SPF/DKIM/DMARC 설정 필수. Resend는 자동 처리
- 개인화 오류 (잘못된 회사명 등): 첫 4주 사람 검수 단계 유지
- 스팸 신고: 시퀀스 unsubscribe 링크 필수

## 9. 프롬프트

`prompts/`:
- `lead-score-classify.md`
- `welcome-email-personalized.md`
- `case-study-pick.md`
