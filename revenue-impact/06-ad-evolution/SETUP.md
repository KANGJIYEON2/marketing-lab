# 06 광고 진화 — 세팅 가이드

`workflows/ad-evolution.json`을 임포트해서 매주 일요일 22시 광고 성과 → 신규 카피 10개 자동 준비.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) + 광고 운영 중

**소요 시간**: 1~2시간

---

## 0. 준비물 체크

- [ ] Layer 0 + `ContentDrafts` + `CampaignMetrics` DB
- [ ] **Meta 광고 운영 중** (월 광고비 $500+ 권장, 데이터 양이 의미 있어야)
- [ ] `META_GRAPH_ACCESS_TOKEN` (03·04에서 발급한 것 + `ads_read` 권한 추가 필요)
- [ ] Meta 광고 계정 ID

---

## 1. Meta Marketing API 권한 추가

03·04에서 만든 Meta App에 광고 권한 추가.

### 1.1 권한 신청

1. https://developers.facebook.com → My Apps → 본인 앱 → App Review → Permissions
2. 다음 권한 신청:
   - `ads_read` — 광고 성과 조회
   - `ads_management` (v2) — 자동 업로드용
3. App Review 통과 전에는 본인 광고 계정으로만 작동. Roles → Testers에 본인 추가.

### 1.2 토큰 권한 확장

기존 token에 새 scope가 포함되어 있는지 확인:

```bash
curl "https://graph.facebook.com/v19.0/me/permissions?access_token={TOKEN}" | jq
```

`ads_read`가 없으면 Graph API Explorer에서 새 토큰 발급:
- Permissions 체크박스에서 `ads_read` 선택
- 토큰 발급 → 장기 토큰 교환 → `.env`의 `META_GRAPH_ACCESS_TOKEN` 업데이트

### 1.3 Account ID 확인

Meta Ads Manager URL: `business.facebook.com/adsmanager/manage/campaigns?act={ACCOUNT_ID}`

또는:
```bash
curl "https://graph.facebook.com/v19.0/me/adaccounts?access_token={TOKEN}"
```

`act_` 접두사 **없이** 숫자만 `.env`에:
```bash
META_ADS_ACCOUNT_ID=1234567890
```

워크플로 URL에서 `act_{{ $json.account_id }}`로 자동으로 접두사 추가됨.

---

## 2. ContentDrafts·CampaignMetrics DB 확인

ContentDrafts 스키마 업데이트 됐는지 확인:
- `channel` Select에 **`meta_ad`** 옵션 있어야 함
- `source_type` Select에 **`ad_pattern`** 옵션 있어야 함
- 신규 속성: `cta` (Rich text), `pattern_used` (Rich text) — 추가했는지

[스키마 문서](../../docs/notion-schemas/content-drafts.md) 참고.

CampaignMetrics DB:
- `channel` Select에 `meta_ads` 있어야 함 (기존 스키마에 이미 포함)
- 모든 필수 속성 (spend·impressions·clicks·conversions·ctr·cpa·roas) 확인

---

## 3. ad_name 명명 규칙 (선택, 권장)

이 자동화의 핵심 한계: **Meta Insights API는 광고 카피 본문을 직접 주지 않음**. ad_name과 adset_name만 보임.

따라서 ad_name을 의미 있게 짓는 규칙이 있어야 LLM이 패턴 추출 가능:

### 권장 명명 규칙

```
{hook}-{audience}-{variant}
```

예시:
- `5min-setup-founder-v1`
- `roas-claim-saas-v2`
- `sunday-savings-marketer-v3`

### 한국어 권장

```
{후킹앵글}-{타겟}-{버전}
```

예시:
- `5분-1인창업자-v1`
- `시간절약-마케터-v2`
- `사회증명-에이전시-v3`

기존 광고들의 이름을 차차 이 규칙으로 바꾸면 다음 주 분석부터 품질 향상.

### 이름 안 바꾸고 본문까지 분석하려면 (v2)

`Meta: Ad Insights` 노드 다음에 추가 분기로 각 ad_id의 creative 본문 호출:

```
GET /{ad_id}?fields=creative{body,title,call_to_action_type}
```

비용: ads × 1 추가 호출. 데이터 더 풍부.

---

## 4. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/ad-evolution.json`

### 임포트 후 확인

- [ ] **Meta: Ad Insights**: URL에 `act_` 접두사 자동, env var 참조 확인
- [ ] **Build Metrics Rows / Notion: Log Metrics**: CampaignMetrics DB
- [ ] **Claude: Extract Patterns / Generate Variants**: Anthropic credential
- [ ] **Fetch Recent Blog Posts**: ContentDrafts DB (filter: status=published, channel=blog)
- [ ] **Notion: Create Variant**: ContentDrafts DB
- [ ] **Slack: Variants Ready**: Slack channel `$env.SLACK_CHANNEL_LOG`
- [ ] **Config** 노드:
  - `our_product_pitch` 본인 제품으로 수정
  - `variants_count` 기본 10 (적절)
  - `top_percent` 0.2 (20%)

---

## 5. 첫 실행 (수동 테스트)

`Sunday 22:00` 트리거 → **Execute Workflow**.

### 통과 시그널

| 노드 | 확인 |
|---|---|
| Meta: Ad Insights | `data` 배열에 ads (지난 7일) |
| Top/Bottom 20% | top/bottom 배열 + week_ending |
| Notion: Log Metrics | CampaignMetrics에 각 ad 행 생성 |
| Claude: Extract Patterns | JSON `{hook_patterns, cta_patterns, ...}` |
| Fetch Recent Blog Posts | 최근 발행 글 최대 5개 |
| Claude: Generate Variants | JSON `{variants: [10개]}` |
| Notion: Create Variant | ContentDrafts에 10개 행 (channel=meta_ad, status=review) |
| Slack | Top 3 variant 미리보기 메시지 |

### 데이터 부족 시

- 광고가 7일 동안 < 5개 → Top/Bottom 20%가 의미 없음 → 모든 광고가 같은 그룹
- spend가 모두 < $1 → ROAS 계산 부정확
- 광고 운영 1개월 이상 후 본 자동화 활성화 권장

---

## 6. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| Meta Insights 401 | 토큰 미발급 또는 `ads_read` 누락 | 1.2 단계 재실행 |
| Meta Insights 빈 결과 | account_id 오류 또는 광고 없음 | Ads Manager URL의 act_ 뒤 숫자 확인 |
| `purchase_roas` 항상 0 | Meta Pixel 미설치 또는 전환 추적 안 됨 | Meta Pixel 설치 + 전환 이벤트 설정 |
| Claude pattern 출력 비어있음 | ad_name이 무의미한 ID | 3번 단계 — ad_name 명명 규칙 도입 |
| variants 10개 미만 생성 | LLM이 가드레일로 일부 폐기 | 정상. 통과한 것만 적재 |
| ContentDrafts 적재 시 `channel` 오류 | `meta_ad` 옵션 미생성 | DB에 옵션 추가 |
| Slack 메시지 너무 김 | variants가 너무 김 | Build Slack Digest의 slice 조정 |

---

## 7. 활성화

수동 테스트 통과 후 토글 **Active**. 매주 일요일 22시 자동.

월요일 아침 9시쯤 Slack에 다이제스트가 와있고, Notion ContentDrafts에 10개 변형이 `status=review`로 준비됨.

---

## 8. 운영 흐름

### 주간 검수 흐름

1. 월요일 출근 → Slack에서 Top 3 미리보기 확인
2. Notion ContentDrafts → filter `channel=meta_ad AND status=review` 진입
3. 10개 변형 검토 — [`prompts/ad-guardrails.md`](./prompts/ad-guardrails.md) 체크리스트
4. 좋은 변형 3~5개 선택 → `status=approved`
5. Meta Ads Manager에 직접 업로드 (MVP는 수동)
6. 2주 후 성과 측정 → 좋은 변형은 시드로 자연스럽게 사용됨 (다음 주 Top 20% 진입)

### 광고 피로도 분산

- 한 광고세트당 변형 3~5개 유지 (다 한꺼번에 활성화 X)
- 매주 1~2개 신규 추가, 가장 오래된 변형 비활성화

### 변형 너무 많이 만들면 안 됨

10개는 검수 가능 한도. 더 만들면:
- 검수 부담 ↑
- 광고세트당 너무 많이 활성화 → 학습 분산
- variants_count Config로 조절

### 성과 추적 (CampaignMetrics)

매주 일요일에 모든 광고의 weekly aggregate가 CampaignMetrics에 적재됨. 09 주간 리포트가 이걸 읽어 트렌드 분석.

### 비용 추정

| 항목 | 비용 |
|---|---|
| Meta API | 무료 (한도 내) |
| Claude pattern + variants | 주 ≈ $0.30 = 월 $1.3 |
| **합계** | **약 $1.3/월** |

---

## 9. 다음 단계

- 첫 2주 변형 채택률 측정 (10개 중 몇 개를 실제 광고로?)
- 채택률 < 30% → 프롬프트 보강, ad_name 명명 규칙 도입
- 채택률 > 60% → v2 자동 업로드 검토
- 다른 자동화 ([09 주간 리포트](../../ops-reduction/09-weekly-report/) — Week 6) 또는 v2 확장
