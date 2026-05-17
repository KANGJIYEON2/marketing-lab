# 07 리드 시퀀스 — 세팅 가이드

`workflows/lead-sequence.json`을 임포트해서 폼 제출 → 5분 이내 개인화 환영 메일.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) 완료 + `Leads` DB 생성

**소요 시간**: 2~3시간 (도메인 인증이 가장 오래)

---

## 0. 준비물 체크

- [ ] Layer 0 + [Leads DB](../../docs/notion-schemas/leads.md)
- [ ] **소유 도메인** — 메일 발신용 (`hello@yourdomain.com` 같은 형태)
- [ ] Apollo.io 계정 (무료 tier OK, 월 50 enrich)
- [ ] Resend 계정 (https://resend.com — 무료 3k 메일/월)
- [ ] 폼 도구 — Tally / Typeform / 또는 자체 LP

---

## 1. Resend 셋업 + 도메인 인증

**가장 오래 걸리는 단계**. DNS 전파 시간 때문에 30분~수 시간 소요.

### 1.1 도메인 추가

1. https://resend.com → Sign up
2. Dashboard → **Domains** → **Add Domain**
3. 발신 도메인 입력: `yourdomain.com` (서브도메인 권장: `mail.yourdomain.com`)
4. Resend가 5개 DNS 레코드 표시 (SPF, DKIM × 2, MX, DMARC)

### 1.2 DNS에 레코드 추가

본인 DNS 관리 (Cloudflare/Gabia/AWS Route 53) 들어가서 Resend가 표시한 5개 레코드 그대로 추가.

| 레코드 | 타입 | 목적 |
|---|---|---|
| `send` 서브도메인 SPF | TXT | "v=spf1 include:amazonses.com ~all" |
| `resend._domainkey` | TXT | DKIM 공개키 1 |
| `_dmarc` | TXT | "v=DMARC1; p=none;" |
| MX | MX | bounce 처리 |

⚠️ Cloudflare 사용 시 **Proxy를 OFF (회색 구름)**로 설정 — 메일 레코드는 프록시되면 안 됨.

### 1.3 인증 확인

Resend Dashboard에서 도메인 상태가 **Verified**로 바뀔 때까지 대기 (5분~수 시간).

### 1.4 API 키 발급

Dashboard → API Keys → **Create API Key** → "Full access"로 발급. `.env`에:
```bash
RESEND_API_KEY=re_...
RESEND_FROM_EMAIL=hello@yourdomain.com
RESEND_FROM_NAME=홍길동
```

`RESEND_FROM_EMAIL`은 위에서 인증된 도메인의 주소여야 함. 별칭(`hello@`, `team@`) 자유.

---

## 2. Apollo.io 셋업

리드 이메일 → 회사·직책·도메인 enrichment.

### 2.1 계정 + API 키

1. https://app.apollo.io → Sign up (무료)
2. Settings → API → **Create API Key**
3. `.env`:
```bash
APOLLO_API_KEY=...
```

### 2.2 무료 tier 한계

- 월 50 credits (= 50 enrich)
- 초과 시 결제 또는 v2에서 Clearbit/Hunter로 fallback

---

## 3. Notion `Leads` DB 확인

[`Leads` 스키마](../../docs/notion-schemas/leads.md) 모든 속성 정확히. 특히:
- `email` (Email)
- `score` (Number)
- `hot_signal` (Checkbox)
- `persona` (Select: founder/marketer/dev/other)
- `sequence_status` (Select: enrolled/day0_sent/...)

Integration 연결 + `.env`:
```bash
NOTION_DB_LEADS=<32자 hex>
```

---

## 4. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/lead-sequence.json`

### 임포트 후 확인

- [ ] **Form Webhook** 노드 URL 복사 → 폼 도구의 webhook destination에 입력 (이전 단계)
- [ ] **Apollo: Enrich** API 키 환경변수 참조
- [ ] **Claude: Score + Welcome Email**: Anthropic credential
- [ ] **Notion: Create Lead / Mark day0_sent**: Notion credential + DB env var
- [ ] **Resend: Send Email**: 환경변수 다 채워졌는지 (API_KEY, FROM_EMAIL, FROM_NAME)
- [ ] **Slack: Hot Lead Alert**: Slack credential + 채널
- [ ] **Welcome Email 프롬프트**의 `Resource pool` URL을 본인 실제 자료 URL로 수정 (현재는 placeholder)

---

## 5. 폼 도구 셋업 (Tally 예시)

### Tally

1. https://tally.so → 폼 생성
2. 필드: `name`, `email` (필수), `company`, `role`, `message` (선택)
3. **Integrations → Webhooks** → URL에 위 4번에서 복사한 webhook URL
4. Trigger: "When form is submitted"
5. Save

### Typeform

비슷한 절차. Webhook을 form Settings → Connect → Webhooks.

### 자체 LP

폼 submit 시 `fetch(WEBHOOK_URL, { method: 'POST', body: JSON.stringify({...}) })` 호출하는 JS 추가. CORS 필요 시 n8n webhook에 CORS 헤더 응답 추가 검토.

---

## 6. 첫 실행 (테스트 제출)

### 방법 A: 폼으로 실제 제출

폼 URL 열어서 본인 이메일로 제출 → 5분 내 메일 도착해야 함.

### 방법 B: curl로 직접 webhook 호출

```bash
curl -X POST {WEBHOOK_URL} \
  -H "Content-Type: application/json" \
  -d '{
    "name": "테스트 사용자",
    "email": "your-test-email@gmail.com",
    "company": "Acme Co",
    "role": "Founder",
    "message": "데모 가능할까요?"
  }'
```

### 통과 시그널

| 단계 | 확인 |
|---|---|
| Form Webhook | n8n에서 실행 트리거됨 |
| Apollo: Enrich | 회사 데이터 또는 실패 (continueOnFail) |
| Claude: Score | JSON `{score, persona, hot_signal, reasoning}` |
| Notion: Create Lead | DB에 새 행 생성 |
| Claude: Welcome Email | JSON `{subject, body_html, body_text}` |
| Resend: Send | response.id 있으면 발송됨 |
| 받은편지함 | 메일 도착 (스팸 폴더도 확인) |
| Slack | hot_signal=true일 때만 알림 |

---

## 7. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| Resend 403 / `domain not verified` | 1.3 미완 | DNS 전파 대기 + Resend Dashboard에서 verify |
| 메일이 스팸으로 감 | DKIM/DMARC 부족 | 모든 DNS 레코드 추가 확인. 발신 시 도메인이 인증된 정확한 도메인인지 |
| Apollo 401 | API 키 오류 | 키 재발급 |
| Apollo credit exhausted | 무료 50건 초과 | 결제 또는 v2 fallback |
| Webhook 응답 없음 | n8n Active 토글 OFF | Active로 |
| `Form Webhook → Normalize Lead`에서 email empty | 폼 필드명 다름 | `Normalize Lead` Code 노드의 키 매핑 보강 |
| Resend 422 | from email이 인증 도메인과 다름 | RESEND_FROM_EMAIL 확인 |
| HTML 메일이 깨짐 | Claude가 인라인 style 사용 | 시스템 프롬프트 강조: "no styling" |

---

## 8. 활성화

테스트 통과 후 워크플로 토글 **Active**. Webhook은 활성 상태에서만 트리거됨.

---

## 9. 운영 팁

### 첫 1주 측정

| 지표 | 목표 | 측정 방법 |
|---|---|---|
| 응답 시간 | < 5분 | Notion `created_at`과 메일 도착 시간 비교 |
| 점수 정확도 | hot 리드 미팅 전환 > 30% | 수동 추적 |
| 메일 도달률 | > 95% | Resend Dashboard |
| 답장률 | > 5% | IMAP 또는 수동 |

### 메일 발송 도메인 분리

서브도메인(`mail.yourdomain.com`)으로 발송하면 메인 도메인 reputation 보호. 권장.

### Apollo 비용 최적화

월 리드 > 50건이면 Apollo 유료 ($49/월 ~ 200 credit). 또는:
- **Clearbit** ($99/월~) — Apollo보다 비싸지만 데이터 품질 ↑
- **Hunter.io Email Finder** ($49/월~) — 기본만 있으면 OK
- **자체 enrich** — 도메인 → 회사명·산업은 LinkedIn 스크래핑 (Phantombuster) 등

### 답장 처리 (v2)

현재 MVP는 답장 감지 없음. 수동 운영:
- Resend Inbox 또는 발신 메일 클라이언트에서 답장 확인
- Notion `Leads`에서 해당 행 `sequence_status = replied`로 수동 변경

v2:
1. **Resend Inbound** 또는 **IMAP** webhook으로 답장 감지
2. Claude로 답장 의도 분류 (예약/관심 표현/거절)
3. Notion 자동 업데이트 + Slack 알림

### 시퀀스 확장 (Day 2, Day 5)

PRD `MVP 범위 (제외)` 항목. 별도 워크플로로 추가:
1. Cron(매일 9시) → Notion에서 `sequence_status = day0_sent AND day0_sent_at >= 2일 전` 리드 조회
2. [`prompts/case-study-pick.md`](./prompts/case-study-pick.md)로 케이스 선택
3. Day 2 메일 작성·발송 → `day2_sent`로 업데이트
4. 같은 패턴으로 Day 5

### 비용 추정 (월 500 리드 기준)

| 항목 | 비용 |
|---|---|
| Claude (스코어링 + 메일) | ≈ $7.5/월 |
| Apollo (50 free, 그 후 $49/월~) | $0~49 |
| Resend (3k free) | $0 |
| **합계** | **$8 ~ $57/월** |

---

## 10. 다음 단계

- 첫 1주 답장률·미팅 전환률 측정
- 답장률 < 3%면 → `welcome-email-personalized.md` 프롬프트 보강 + 도메인 reputation 점검
- 답장률 > 8%면 → v2 시퀀스(Day 2/5) 우선순위 ↑
- 또는 다음 자동화([05 SEO 헌터](../../content-machine/05-seo-hunter/) — Week 5)로 이동
