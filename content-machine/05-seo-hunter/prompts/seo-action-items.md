# seo-action-items

내 글 본문 + Top 10 SERP 결과 → SERP 요약·약점·결손 섹션·구체 액션 아이템.

## 모델

- 권장: `claude-sonnet-4-6` (긴 컨텍스트 필요)
- temperature: 0.3 (분석은 결정론적)
- max tokens: 1200

## 시스템 메시지

```
You analyze why a page is stuck on page 2 of Google and propose specific updates to push it to page 1.

Given:
- My page text (currently positioned 11-20 for the target keyword)
- Top 10 SERP results (titles + snippets)

Return STRICT JSON:
{
  "serp_summary": "<2-3 sentences on what Top 10 results emphasize/cover>",
  "missing_sections": ["<section topic 1>", "<section topic 2>", "..."],
  "weak_points": ["<weakness 1>", "..."],
  "action_items": ["<concrete action 1>", "<concrete action 2>", "..."]
}

Action item rules:
- Each item starts with a verb (Add, Replace, Expand, Rewrite, Restructure, Include, Update)
- Reference specific content (e.g., "Add a 'pricing comparison' table covering X, Y, Z" not just "Add comparison")
- 3-5 items max — prioritize impact over quantity
- Skip generic SEO advice ("add internal links", "improve speed")

No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
Target keyword: {keyword}
Current position: {position}
Impressions (28d): {N}
CTR: {N}

My page URL: {url}
My page text (truncated to 6000 chars):
---
{my page text}
---

Top 10 SERP results:
{JSON array with position/title/snippet/link}
```

## 출력 예시

### Case 1 — 비교 글, 비교표가 빠짐
입력 키워드: "n8n vs zapier"

```json
{
  "serp_summary": "Top 10 글들은 모두 가격·통합 수·AI 기능을 표로 비교하고, 'when to choose each' 의사결정 가이드를 포함. 셀프호스팅 가능 여부를 핵심 차별점으로 강조.",
  "missing_sections": [
    "가격 플랜 직접 비교표 (3개 tier)",
    "통합 가능 앱 수 비교 (8000+ vs 3000+)",
    "셀프호스팅 가능 여부 + 이유",
    "Decision framework: 어떤 상황에 어떤 도구"
  ],
  "weak_points": [
    "본문이 n8n 장점에만 치우쳐 zapier 비교 부족",
    "구체적 가격 수치 부재",
    "FAQ 섹션 없음 (Top 10 중 7개 포함)"
  ],
  "action_items": [
    "Add a 3-column pricing comparison table (Free / Starter / Business tiers for both)",
    "Add a 'Self-hosted vs Cloud' section highlighting n8n's unique advantage",
    "Expand the integration count comparison with specific examples (Slack, Google Sheets, Notion)",
    "Replace the conclusion with a decision framework: 'Pick Zapier if X / Pick n8n if Y'",
    "Add FAQ section covering: pricing surprises, migration difficulty, learning curve"
  ]
}
```

### Case 2 — How-to 글, 단계 부족
입력 키워드: "n8n self-host docker"

```json
{
  "serp_summary": "Top 10은 모두 step-by-step 명령어 + 환경변수 설명 + post-install 검증 포함. 'gotchas' 섹션이 거의 모든 글에 있음.",
  "missing_sections": [
    "Post-installation health check 명령어",
    "Common errors and fixes (port already in use, permissions)",
    "Backup strategy",
    "Upgrade procedure"
  ],
  "weak_points": [
    "설치까지만 다루고 운영 단계 부재",
    "에러 시 디버깅 가이드 없음"
  ],
  "action_items": [
    "Add a 'Verify Installation' section with 3 curl commands to confirm n8n is running",
    "Add 'Common Errors' table: port conflict, db permissions, webhook URL misconfig — each with fix",
    "Add 'Backup & Upgrade' section: how to backup .n8n/ volume + safely upgrade Docker image",
    "Include a real `.env` example with production-grade settings (HTTPS, encryption key)"
  ]
}
```

### Case 3 — 1차 분석이 효과 적은 케이스 (이미 잘 작성됨)
```json
{
  "serp_summary": "Top 10 결과가 모두 매우 유사 — 표면적인 정의 + 5가지 예시. 차별화 어려움.",
  "missing_sections": [],
  "weak_points": [
    "특별히 약하지 않음. 1페이지 진입은 콘텐츠 보강이 아닌 백링크/내부링크 강화로 가능"
  ],
  "action_items": [
    "Add 3-5 internal links from your high-traffic posts pointing to this page",
    "Consider creating a related supporting article and link to this one as the pillar",
    "Skip content rewrite — current quality matches Top 10"
  ]
}
```

## 평가 기준

- [ ] `action_items`가 매크로가 아닌 구체적 (테이블/섹션/예시 명시)
- [ ] 동사로 시작 (Add/Replace/Expand 등)
- [ ] 3~5개 — 너무 많으면 우선순위 모호
- [ ] 일반적 SEO 조언 ("Add internal links" 같은) 비율 < 20%
- [ ] `missing_sections`가 실제 Top 10에 있는 토픽과 일치 (LLM이 상상한 게 아님)

## 흔한 LLM 실수

1. **일반 SEO 조언 남발** — "메타 디스크립션 개선", "이미지 alt 태그 추가". 이런 건 GSC 헌터의 본질 아님
2. **추측 기반 missing_sections** — Top 10이 실제로 다루지 않는 토픽을 "있는 척" 함
3. **action_items가 모호** — "Improve content quality" 같은 액션 아닌 액션
4. **5개를 넘김** — 더 많이 = 더 좋은 것 아님. 영향 큰 것 3개가 진리

## 한국어 SEO에서

- SERP가 한국어면 출력도 한국어로 (시스템에 "Output language: match keyword language" 추가 검토)
- 네이버는 GSC와 무관. 본 자동화는 Google 검색만 다룸
- 네이버용은 별도 도구 필요 (네이버 검색광고 API + 키워드 도구)

## 비용

- 입력: ~5k tokens (my_text 6k + SERP + system)
- 출력: ~700 tokens
- 기회 1건당 ≈ $0.025
- 주 10건 처리 = 월 $1
