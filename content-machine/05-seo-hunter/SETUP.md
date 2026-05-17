# 05 SEO 헌터 — 세팅 가이드

`workflows/seo-hunter.json`을 임포트해서 매주 월요일 6시에 GSC 2페이지 키워드 자동 발굴.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) 완료 + `SEOOpportunities` DB

**소요 시간**: 1~2시간 (GSC 권한 + Serpapi 셋업)

---

## 0. 준비물 체크

- [ ] Layer 0 + [SEOOpportunities DB](../../docs/notion-schemas/seo-opportunities.md)
- [ ] **Google Search Console에 등록된 사이트** (소유권 확인 완료)
- [ ] 발행된 글 **10편 이상** (이하면 의미 있는 11-20위 키워드 없음)
- [ ] Google Cloud 프로젝트 (03 통합 인박스에서 만든 것 재사용 가능)
- [ ] Serpapi.com 계정 (무료 100 검색/월)

---

## 1. Google Search Console 권한 (n8n OAuth)

### 1.1 Search Console API 활성화

1. https://console.cloud.google.com → 03에서 만든 프로젝트 (또는 신규)
2. APIs & Services → Library → **Search Console API** 검색 → Enable

### 1.2 OAuth Scope 추가

이미 03에서 Gmail OAuth 셋업했다면 동일 OAuth client에 scope 추가:

1. APIs & Services → OAuth consent screen → Edit App → Scopes
2. **추가**: `https://www.googleapis.com/auth/webmasters.readonly`
3. Save

### 1.3 n8n Google API Credential

1. n8n → Credentials → **Add Credential** → **Google API**
2. Service Account 또는 OAuth2 둘 다 가능 — OAuth2 권장 (간단)
3. Client ID / Secret 입력 (03에서 만든 OAuth client 재사용)
4. Scope: `https://www.googleapis.com/auth/webmasters.readonly`
5. **Sign in with Google** → 권한 승인

### 1.4 GSC site URL 확인

GSC에 등록된 사이트의 정확한 URL을 `.env`에:

```bash
GSC_SITE_URL=https://yourdomain.com/
```

⚠️ Domain property (`sc-domain:yourdomain.com`)인지 URL prefix (`https://yourdomain.com/`)인지에 따라 형식 다름:
- URL prefix: `https://yourdomain.com/` (trailing slash 포함)
- Domain property: `sc-domain:yourdomain.com`

GSC 좌상단 드롭다운에서 본인 site 정확히 복사.

---

## 2. Serpapi 셋업

### 2.1 계정 + API 키

1. https://serpapi.com → Sign up (무료 100 검색/월)
2. Dashboard → API Key 복사
3. `.env`:
```bash
SERPAPI_KEY=...
```

### 2.2 무료 tier 운영

- 주 10건 분석 × 4주 = 월 40 검색 → 무료 tier 충분
- 초과 시 $50/월부터 (5k 검색)

### 대안

비용 부담되면:
- DataForSEO ($25/월 prepaid)
- ScraperAPI (구글 SERP 미공식 지원)
- **MVP에서 SERP 호출 제거**: Claude에 my_text만 주고 일반론 액션. 품질 떨어지지만 비용 0.

SERP 제거 옵션: `SERP: Top 10` 노드 비활성화 + `Bundle Context` Code 노드의 `serp_results`를 빈 배열로 처리.

---

## 3. Notion `SEOOpportunities` DB

[`SEOOpportunities` 스키마](../../docs/notion-schemas/seo-opportunities.md) 모든 속성 확인. 특히:
- `url` (URL)
- `keyword` (Rich text)
- `position`, `impressions`, `clicks`, `ctr` (Number)
- `priority_score` (Number)
- `action_items` (Rich text)
- `status` (Select: new/triaged/in_progress/done/archived)

Integration 연결 + `.env`:
```bash
NOTION_DB_SEO_OPPORTUNITIES=<32자 hex>
```

---

## 4. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/seo-hunter.json`

### 임포트 후 확인

- [ ] **GSC: Search Analytics**: Google API credential 선택, URL의 site_url encodeURI 자동 처리
- [ ] **Already Tracked? / Notion 노드들**: Notion credential
- [ ] **Claude: SEO Action Items**: Anthropic credential
- [ ] **Slack: SEO Digest**: Slack credential + 채널 (`$env.SLACK_CHANNEL_LOG`)
- [ ] **Config** 노드 값 확인:
  - `min_impressions`: 100 (너무 작은 키워드 노이즈 제거)
  - `top_n_for_serp`: 10 (LLM·SERP 호출 비용 통제)
  - `avg_top10_ctr`: 0.12 (1페이지 평균 CTR 가정)

---

## 5. 첫 실행 (수동 테스트)

`Weekly Mon 06:00` 트리거 → **Execute Workflow**.

### 첫 실행의 의미

GSC 데이터는 지난 28일 누적. 처음 실행해도 데이터가 잡힘 (사이트가 등록되어 있고 클릭이 있다면).

### 통과 시그널

| 노드 | 확인 |
|---|---|
| GSC: Search Analytics | `rows` 배열에 키워드 데이터 |
| Filter Opportunities | position 11-20, impressions ≥ 100 필터 통과 |
| Top N for SERP | 상위 10개로 제한 |
| Fetch My Page | HTML 응답 |
| SERP: Top 10 | `organic_results` 배열 |
| Bundle Context | my_text + serp_results 합쳐짐 |
| Claude: SEO Action Items | JSON 응답 |
| Notion: Add Opportunity | DB에 행 생성 + 본문 블록까지 |
| Slack: SEO Digest | 채널에 다이제스트 도착 |

---

## 6. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| GSC 403 | OAuth scope 누락 또는 site 권한 없음 | 1.2~1.4 재확인, GSC에서 본인이 site owner인지 확인 |
| GSC 404 | site_url 형식 오류 | URL prefix vs domain property 정확히 |
| `Filter Opportunities` 0건 | min_impressions가 너무 높음 또는 신규 사이트 | 임계값을 50으로 낮춰 테스트 |
| SERP 401 | API 키 오류 | Serpapi Dashboard에서 키 재확인 |
| SERP 429 | 무료 tier 초과 | 다음 달까지 대기 또는 유료 |
| Fetch My Page 403/Cloudflare 차단 | 봇 차단 | User-Agent 헤더 추가 (HTTP Request 노드 수정) |
| `Already Tracked?` 항상 신규로 인식 | Notion title 비교 형식 mismatch | 워크플로 Title 표현식과 DB 페이지 title 일치 확인 |
| Claude action_items가 일반론 | SERP 데이터 부족 또는 my_text 너무 짧음 | prompts 평가 기준에 따라 시스템 프롬프트 보강 |

---

## 7. 활성화

수동 테스트 통과 후 워크플로 토글 **Active**. 매주 월요일 6시 자동 실행.

월요일 6시인 이유: GSC 데이터 지연(2-3일)으로 일요일 데이터 부정확. 월요일 새벽이면 수요일까지 데이터 포함됨.

---

## 8. 운영 팁

### 액션 실행 흐름

1. 월요일 9시 출근 → Slack에 SEO 다이제스트 도착해 있음
2. Notion `SEOOpportunities` DB 열어 Top 5 검토
3. 우선순위 1~2개 선택 → `status = triaged`
4. 이번 주에 글 업데이트 → `status = in_progress`
5. 발행 → `status = done`, `done_at` 자동 기록
6. 4주 후 09 주간 리포트에서 `result_position` 자동 측정 (v2)

### priority_score 활용

- > 1000: 즉시 작업 (큰 임팩트 예상)
- 500-1000: 다음 주 이내
- < 500: 시간 날 때

### Top N 외 기회는 어디로?

워크플로의 별도 분기에서 **LLM 분석 없이 Notion에만 적재** (`Notion: Add (skipped LLM)`). action_items가 placeholder. 다음 주 실행 시 priority_score가 더 높아지면 자연스럽게 Top N 진입 → 그때 LLM 분석.

### 4주 후 측정 (v2)

본 MVP는 추적만. 측정은 09 주간 리포트와 연동:
1. 09가 매주 GSC 호출 시 `SEOOpportunities`의 `status=done AND done_at >= 4주 전` 행 조회
2. 같은 (url, keyword)의 현재 position을 GSC에서 fetch
3. `result_position`에 기록 → win rate 자동 계산

### 비용 추정

| 항목 | 주 10건 기준 |
|---|---|
| GSC API | 무료 |
| Serpapi | 무료 tier 내 |
| Fetch My Page | 무료 |
| Claude | ≈ $0.25/주 = 월 $1 |
| **합계** | **$1/월** |

---

## 9. 다음 단계

- 첫 1주 액션 아이템 품질 점검 → 일반론 비율 높으면 [`prompts/seo-action-items.md`](./prompts/seo-action-items.md) 평가 기준대로 프롬프트 보강
- 4주 후 첫 measurement → 액션 실행한 글의 position 변화 추적
- 다음 자동화 ([06 광고 진화](../../revenue-impact/06-ad-evolution/) — 같은 Week 5) 또는 [09 주간 리포트](../../ops-reduction/09-weekly-report/) — 다른 자동화 결과 통합 시작
