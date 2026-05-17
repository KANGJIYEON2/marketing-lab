# 10 LP A/B — 세팅 가이드

워크플로 2개로 분리: `generate-variants.json` (webhook으로 실험 시작) + `check-results.json` (cron으로 매일 측정·유의성 계산).

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) + `CampaignMetrics` DB

**소요 시간**: 2~3시간 (LP 플랫폼별 배포 방식 학습 포함)

---

## 0. 준비물 체크

- [ ] Layer 0 + CampaignMetrics
- [ ] **GA4 측정 ID + Custom dimension 등록 가능 권한**
- [ ] LP 플랫폼 (Webflow / Framer / Next.js / WordPress 등) + 변형 배포 방법
- [ ] 일평균 LP 트래픽 **100+** (그 이하면 실험 무의미)

---

## 1. GA4 Custom Dimension 등록

본 자동화의 핵심: GA4가 각 방문자가 어떤 variant를 봤는지 트래킹하려면 custom dimension 2개 필요.

### 1.1 GA4 Admin → Custom definitions

1. https://analytics.google.com → 본인 GA4 property
2. Admin → Data display → **Custom definitions** → **Create custom dimension**
3. **2개 등록**:

| Dimension name | Scope | Event parameter | 설명 |
|---|---|---|---|
| `experiment_id` | Event | `experiment_id` | 어떤 실험 |
| `variant_id` | Event | `variant_id` | 어떤 변형 (control / A / B...) |

### 1.2 LP에서 이벤트 발송

LP가 로드될 때 + 전환 이벤트 발생 시 두 dimension을 GA4에 보내야 함.

#### Next.js / 자체 LP (가장 유연)

`_app.js` 또는 layout:
```js
// 방문자 진입 시 variant 할당 (cookie or URL param)
const variantId = getVariantFromCookie() || assignVariant(['control', 'A']);
setCookie('variant_id', variantId);

// GA4로 view event 발송
gtag('event', 'lp_view', {
  experiment_id: 'exp_2026-05-17_hero-headline',
  variant_id: variantId,
});

// 전환 시
gtag('event', 'lp_conversion', {
  experiment_id: 'exp_2026-05-17_hero-headline',
  variant_id: variantId,
});
```

#### Webflow

1. Site settings → Custom code → Head:
```html
<script>
  // experiment routing logic
  if (!localStorage.getItem('variant_id')) {
    const v = Math.random() < 0.5 ? 'control' : 'A';
    localStorage.setItem('variant_id', v);
  }
  const variant = localStorage.getItem('variant_id');

  // GA4 event
  gtag('event', 'lp_view', {
    experiment_id: 'exp_2026-05-17_hero-headline',
    variant_id: variant,
  });

  // Apply variant: hide control hero, show variant hero
  document.addEventListener('DOMContentLoaded', () => {
    document.getElementById(`hero-${variant}`).style.display = 'block';
    document.querySelectorAll(`[id^="hero-"]:not(#hero-${variant})`).forEach(el => el.style.display = 'none');
  });
</script>
```

Webflow Designer에서 hero 섹션 N개 (id `hero-control`, `hero-A`, `hero-B`) 만들고 모두 default hidden.

#### Framer

비슷한 방식. Code component 또는 Custom code 활용.

#### WordPress

Plugin (Nelio A/B Testing 등) 사용 권장. Plugin이 자동으로 variant 할당 + GA4 dimension 발송 지원.

---

## 2. GA4 API 권한 (Google API credential)

03·05에서 만든 Google API credential에 GA4 scope 추가:

1. Google Cloud Console → APIs & Services → Library → **Google Analytics Data API** 활성화
2. OAuth Consent Screen → Scopes에 `https://www.googleapis.com/auth/analytics.readonly` 추가
3. n8n Google API credential 재인증 (Sign in with Google → 권한 다시 승인)

### GA4 Property ID

GA4 → Admin → Property → **Property settings** → Property ID (숫자 9-10자리)

`.env`:
```bash
GA4_PROPERTY_ID=123456789
```

---

## 3. CampaignMetrics DB 확인

이미 [`CampaignMetrics`](../../docs/notion-schemas/campaign-metrics.md)에 LP A/B용 필드 모두 있음:
- `channel`: `lp_experiment` 옵션
- `entity_type`: `lp_variant` 옵션
- `experiment_id` (Rich text)
- `winner` (Checkbox)
- `notes` (Rich text — headline 본문 저장용)

추가 작업 없음. Layer 0에서 만들었다면 그대로 사용.

---

## 4. 워크플로 임포트

### 4.1 `generate-variants.json`

n8n → Import → `workflows/generate-variants.json`

- **Experiment Start Webhook** URL 복사 (실험 시작 시 호출용)
- Notion credential
- Anthropic credential
- Slack credential

### 4.2 `check-results.json`

n8n → Import → `workflows/check-results.json`

- Notion credential
- Google API credential (GA4 scope 포함)
- Slack credential

---

## 5. 실험 시작 (`generate-variants` 호출)

curl 또는 Postman으로 webhook 호출:

```bash
curl -X POST {WEBHOOK_URL} \
  -H "Content-Type: application/json" \
  -d '{
    "experiment_id": "exp_2026-05-17_hero-headline",
    "lp_url": "https://yourdomain.com/pricing",
    "control_headline": "마케팅 자동화 솔루션",
    "control_subhead": "1인 창업자를 위한 도구",
    "target_persona": "1인 SaaS 창업자",
    "variants_count": 2,
    "hypothesis": "구체적 시간 약속이 추상적 가치보다 전환 높을 것"
  }'
```

응답:
```json
{ "ok": true, "experiment_id": "exp_2026-05-17_hero-headline" }
```

직후:
- Notion `CampaignMetrics`에 **Control + Variant A·B** 행 N+1개 생성
- Slack에 변형 미리보기 알림

### 5.1 검토 + LP 배포 (수동)

1. Slack 알림에서 변형 확인
2. Notion에서 변형 본문·hook_angle 검토
3. **마음에 드는 변형 1개 선택** (또는 그대로 다 사용)
4. LP에 hero 섹션 N개 추가:
   - `hero-control`: Control headline
   - `hero-A`: Variant A headline
   - 1번 단계의 코드로 무작위 분기
5. LP 배포 (Webflow publish / Framer publish / Next.js deploy)

### 5.2 GA4에서 트래픽 분기 확인

GA4 Realtime → Custom dimensions에 `experiment_id`, `variant_id`가 보이고 50/50 분기되는지 확인.

---

## 6. 결과 확인 (`check-results` 자동)

매일 7시에 자동 실행. 단계:

1. CampaignMetrics에서 `entity_type=lp_variant AND winner=false` 행 fetch
2. experiment_id별 그룹핑
3. GA4 query: 각 variant의 sessions + conversions (30일 lookback)
4. **z-test**: Control vs 각 Treatment
   - 최소 sample (500 per variant)
   - p < 0.05 → 유의 (significant)
5. 유의한 winner 있으면 → Slack 알림

### 통과 시그널

| 노드 | 확인 |
|---|---|
| Fetch Active Experiments | `entity_type=lp_variant AND winner=false` 행 |
| Group by Experiment | experiment_id별 묶음 |
| GA4: Variant Stats | 각 variant의 sessions·conversions |
| Compute Significance | z-test 결과, p-value, lift |
| Has Winner? | 유의한 winner 있을 때만 다음 단계 |
| Slack: Winner Alert | 위너 결정 메시지 |

---

## 7. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| GA4 query 빈 결과 | custom dimension 미등록 또는 LP에서 발송 안 됨 | 1번 단계 재확인. GA4 DebugView로 실시간 확인 |
| GA4 권한 오류 | scope 누락 | 2번 단계 OAuth 재승인 |
| sample size 부족 | 트래픽 100/일 미만 | 실험 기간 늘리거나, 통계적 검증 어려움 인정 |
| 양쪽 사용자가 같은 variant 봄 | localStorage 충돌 | LP의 variant 할당 로직 점검. URL param 방식 권장 |
| `p` 값이 항상 null | sample < min_sample_per_variant (기본 500) | Config의 min_sample 낮추거나 트래픽 누적 대기 |
| Variant lift 음수인데 winner | p<0.05 + lift>0 조건 필터. 음수면 위너 결정 안 됨 | 워크플로 동작 정상 |

---

## 8. 활성화

- `generate-variants.json`: Active 토글 (webhook 활성화)
- `check-results.json`: Active 토글 (매일 7시 cron)

---

## 9. 운영 원칙

### 최소 sample size

본 워크플로는 variant당 500 sessions 미달 시 유의성 계산 skip. 실험 기간 산정:

| 일평균 LP 세션 | 100 | 300 | 1000 | 3000 |
|---|---|---|---|---|
| Variant당 500 도달 (2 variants) | 10일 | 4일 | 1일 | <1일 |
| Variant당 500 도달 (4 variants) | 20일 | 7일 | 2일 | <1일 |

**일평균 100 미만이면 의미 있는 실험 어려움.** 광고로 트래픽 확보 후 실험 권장.

### 위너 결정 후 흐름

`check-results`가 위너 알림 보내면:

1. **사람이 검토** — lift가 예상과 다른지, 어떤 변형이 위너인지
2. Notion에서 위너 variant 행 → `winner=true` 체크
3. LP에 위너 헤드라인 **수동 적용** (control → 위너로 교체)
4. 다음 실험 시작 시 새 control은 이번 위너

### 동시 실험 한도

여러 LP에서 동시 진행 OK. 같은 LP에서는 한 번에 1개 실험 (변수 격리).

### CTA·Social Proof 실험 (v2)

[`prompts/lp-variant-cta.md`](./prompts/lp-variant-cta.md), [`prompts/lp-variant-social-proof.md`](./prompts/lp-variant-social-proof.md) 명세 참고. 헤드라인 실험 안정화 후 추가.

### 비용 추정

| 항목 | 비용 |
|---|---|
| Claude (실험 시작 시 1회) | $0.03/실험 |
| GA4 + Notion | 무료 |
| LP 플랫폼 | 본인 보유 |
| **합계** | **거의 0** |

---

## 10. 다음 단계

- **마지막 자동화** [09 주간 리포트](../../ops-reduction/09-weekly-report/)로 이동 — 다른 모든 자동화 결과를 통합 분석
- 또는 본 자동화의 v2 확장 (CTA → Social proof → 이미지)
