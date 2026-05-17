# 04 경쟁사 인텔 — 세팅 가이드

`workflows/competitor-intel.json`을 임포트해서 매일 8시에 경쟁사 블로그·가격·광고 변화 모니터링.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) 완료 + `CompetitorTimeline` DB 생성

**소요 시간**: 1시간 (경쟁사 정보 조사 시간 별도)

---

## 0. 준비물 체크

- [ ] Layer 0 완료 + [`CompetitorTimeline` DB](../../docs/notion-schemas/competitor-timeline.md) 생성
- [ ] 모니터링할 경쟁사 2~3개 선정
- [ ] 각 경쟁사의 **블로그 RSS URL**, **가격 페이지 URL**, **Meta Page ID** (선택)
- [ ] `META_GRAPH_ACCESS_TOKEN` (이미 03에서 발급했으면 그대로 사용)

---

## 1. 경쟁사 정보 수집

워크플로 `Config` 노드 안에 JSON으로 직접 박는다. 각 경쟁사마다:

### 1.1 블로그 RSS URL 찾기

대부분 사이트에 RSS 있음. 시도 순서:
1. `https://{domain}/feed`
2. `https://{domain}/rss`
3. `https://{domain}/blog/feed`
4. `https://{domain}/blog/rss.xml`
5. 페이지 소스 보기 → `<link rel="alternate" type="application/rss+xml" ...>`

브라우저로 직접 방문해서 XML이 보이면 OK.

### 1.2 가격 페이지 URL

대부분 `/pricing` 또는 `/plans`. 메인 메뉴에서 확인. **이 페이지의 변경이 가장 시그널이 강하다** — 가격, 플랜 구조, 무료 체험 기간이 잘 보이는 URL을 선택.

### 1.3 Meta Page ID (선택)

경쟁사가 Meta(Facebook/IG) 광고를 돌리지 않으면 건너뜀.

찾는 법:
1. 경쟁사 Facebook 페이지 방문
2. About → Page Transparency 또는 페이지 정보
3. **Page ID** 항목에서 숫자 복사

또는 https://www.facebook.com/ads/library/ 에서 검색 → 페이지 클릭 → URL에 `view_all_page_id=...` 파라미터.

---

## 2. Config 노드 채우기

워크플로 임포트 후 `Config (edit competitors)` 노드 클릭 → `competitors` 값 수정:

```json
[
  {
    "name": "Competitor A",
    "domain": "competitor-a.com",
    "blog_rss": "https://competitor-a.com/blog/feed",
    "pricing_url": "https://competitor-a.com/pricing",
    "meta_page_id": "123456789"
  },
  {
    "name": "Competitor B",
    "domain": "competitor-b.com",
    "blog_rss": "https://competitor-b.com/rss",
    "pricing_url": "https://competitor-b.com/plans",
    "meta_page_id": ""
  }
]
```

규칙:
- `name`은 Notion DB의 `competitor` Select 옵션과 정확히 일치
- 사용 안 할 source는 빈 문자열 (`""`) — 워크플로가 skip
- `our_context`도 본인 제품 상황으로 수정 (Claude 분석 정확도에 큰 영향)

---

## 3. Meta Ad Library 접근 (선택)

Meta Ad Library API는 **공개 광고**만 접근 가능. 누구나 access token으로 호출 가능.

이미 03 통합 인박스에서 `META_GRAPH_ACCESS_TOKEN` 발급했으면 그대로 사용. 안 했다면:
1. https://developers.facebook.com/tools/explorer
2. User access token 생성 — 권한은 `ads_archive` 하나만 (공개 API)
3. `.env`에 `META_GRAPH_ACCESS_TOKEN=` 채움

⚠️ Meta Ad Library는 한국 광고는 정치/사회 이슈만 색인. 일반 상업 광고는 미국·EU 등에서만 보임. 한국 시장만 보면 이 source는 효과 제한적 — `meta_page_id`를 비워두면 됨.

---

## 4. Notion DB 확인

[`CompetitorTimeline` 스키마](../../docs/notion-schemas/competitor-timeline.md) 모든 속성·옵션이 정확히 있어야 함.

특히 `competitor` Select 옵션에 위 Config의 `name` 값을 미리 추가:
- `Competitor A`, `Competitor B` 등

자동화 첫 실행 시 Notion이 자동으로 옵션 추가하기도 하지만, 미리 만들어두면 색상 지정·정렬에 좋다.

`.env`:
```bash
NOTION_DB_COMPETITOR_TIMELINE=<32자 hex>
```

---

## 5. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/competitor-intel.json`

### 임포트 후 확인

- [ ] **Fetch Blog RSS / Pricing / Ad Library**: `continueOnFail: true` 설정됨 (한 경쟁사가 죽어도 다른 곳 진행). 그대로 둘 것
- [ ] **Lookup Previous / Notion: Append**: Notion credential 선택, DB 환경변수 참조
- [ ] **Claude: Analyze Change**: Anthropic credential
- [ ] **Slack: #competitor-intel**: Slack credential + 채널 `$env.SLACK_CHANNEL_COMPETITOR`
- [ ] Config 노드의 `competitors` JSON과 `our_context` 수정 완료

---

## 6. 첫 실행 (수동 테스트)

`Daily 08:00` 트리거 노드 → **Execute Workflow**.

### 첫 실행의 의미

**첫 실행 시 모든 가격 페이지가 "신규 스냅샷"**으로 분류됨 (이전 데이터 없음). `is_first_snapshot=true` 케이스 — Claude는 `impact_score: 0`, `our_action: informational only`로 답해야 함. 이게 정상.

블로그·광고도 첫 실행 때 "최근 24시간 내 신규" 필터만 통과한 것만 잡힘.

따라서 첫 실행에서는 의미 있는 알림이 거의 없을 수 있다. **3~7일 굴리면서 진짜 변경이 잡혀야 가치 검증** 가능.

### 통과 시그널

| 노드 | 확인 |
|---|---|
| Split per Competitor | 경쟁사 N개로 split |
| Fetch ... | 각 source 응답 (실패해도 continueOnFail로 통과) |
| Parse Blog / Extract Pricing / Parse Ads | 각각 객체 또는 빈 배열 |
| Merge per Competitor | 모든 source 통합 |
| Lookup Previous | 첫 실행은 빈 결과 |
| Detect Real Change | 신규 또는 변경된 것만 통과 |
| Claude: Analyze Change | JSON 응답 |
| Notion: Append | DB에 행 생성 |
| Slack | change 행만 알림 (snapshot은 skip) |

---

## 7. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| RSS 404 / 응답 비어있음 | URL 오타 또는 사이트가 RSS 미제공 | 브라우저로 직접 확인. 없으면 해당 source 건너뜀 (`continueOnFail` 작동) |
| Pricing diff가 항상 발생 | 페이지에 동적 콘텐츠 (광고, 타임스탬프) | `Extract Pricing Text` Code 노드 수정: CSS selector로 가격 섹션만 추출 (v2) |
| Meta Ad Library 401 | access token 만료 | 토큰 재발급 |
| Meta Ad Library 빈 결과 | 한국 광고는 정치 이슈만 색인 | 정상. 다른 시장 광고면 잡힘 |
| Notion `select option does not exist` | `competitor` 옵션 미생성 | 위 4번 단계대로 미리 옵션 추가 |
| Claude impact_score 모두 6+ | LLM이 모든 변경을 중요하다고 판단 | 시스템 프롬프트의 impact scale 강조, 첫 1주 매일 점검 후 보정 |

---

## 8. 활성화

수동 테스트 통과 후 워크플로 토글 **Active**. 매일 8시 자동 실행.

⚠️ 너무 자주 폴링하지 말 것. RSS·HTML scraping은 일 1회면 충분. 시간별 폴링은 사이트에 부담 + IP 차단 위험.

---

## 9. 운영 팁

### 첫 2주 점검 루틴

매일 Slack `#competitor-intel` 알림을 보면서:
- [ ] `impact_score`가 합리적인지 (모두 5+면 LLM이 보정 필요)
- [ ] `our_action`이 실행 가능한지 (추상적이면 [`prompts/competitor-action-suggest.md`](./prompts/competitor-action-suggest.md) 가이드 강화)
- [ ] 실제 실행한 액션 비율 측정 (월말)

### 가격 페이지 diff 노이즈 줄이기

가격 페이지에 자주 바뀌는 동적 요소(쿠키 배너, 광고, A/B 테스트)가 있으면 매일 diff 발생. 해결:
1. `Extract Pricing Text` Code 노드의 정규식을 수정해 특정 selector만 추출
2. 또는 페이지 안의 "가격" 키워드 주변 텍스트만 캡처
3. 또는 가격 페이지 대신 가격이 적힌 페이지(예: `/plans` 또는 FAQ) 사용

### 경쟁사 추가

`Config` 노드의 `competitors` 배열에 새 객체 추가. Notion `competitor` Select에도 새 옵션 추가.

권장: 직접 경쟁자 3~5개에 집중. 너무 많으면 노이즈 ↑ + LLM 비용 ↑.

### v2 추가 후보

PRD `MVP 범위 (제외)` 항목:
1. **SNS 모니터링** — IG/X/LinkedIn 게시물 변화
2. **Product Hunt / Hacker News 멘션** — 경쟁사가 새 제품 런칭하면 알림
3. **자동 액션 → 자동화 실행 연결** — 예: 새 콘텐츠 발견 시 [01](../../content-machine/01-idea-engine/) 시드로 자동 추가

### 비용 추정

| 항목 | 일 변경 평균 3건 기준 |
|---|---|
| Claude API | $0.024/일 ≈ $0.75/월 |
| Notion / Slack | 무료 |
| HTTP fetch | 무료 |
| **합계** | < **$1/월** |

매우 저렴. 가장 큰 가치는 사람 시간 절약(주 2~3시간).

---

## 10. 다음 단계

- 첫 1주 결과 보고 가격 페이지 diff 노이즈 보정
- 실행한 액션 누적 → 좋은 예시들을 `competitor-action-suggest.md`에 few-shot으로 추가 (시스템 프롬프트 강화)
- 다른 자동화 ([07 리드 시퀀스](../../revenue-impact/07-lead-sequence/) — 같은 Week 4)로 이동
