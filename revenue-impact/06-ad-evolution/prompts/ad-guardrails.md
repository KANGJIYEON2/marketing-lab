# ad-guardrails

광고 카피가 Meta·Google 광고 정책 + 자사 브랜드 가드레일을 위반하지 않도록 하는 룰 모음. 별도 LLM 호출이 아니라 `ad-variant-generate.md` 시스템 메시지의 일부.

## Meta 광고 정책 (위반 시 광고 reject)

### 절대 금지
- 차별 표현 (인종·종교·성별·나이 기반 타겟팅 암시)
- 개인 속성 단정 ("당신은 우울하군요" 같은)
- 무허가 제약·금융 상품
- 비현실적 결과 약속 ("100% 보장")
- before/after 비교 (특히 신체)

### 제한
- 의료·금융·구직 카테고리는 별도 승인 필요
- 알코올·도박 — 지역별 제한

## Google Ads 정책 (v2)

- Search Network는 본문이 더 짧음 (description 90자)
- Display Network는 이미지 기반이라 카피보다 visual 중심
- 본 자동화 MVP는 Meta만. Google은 v2.

## 자사 브랜드 가드레일

### 카피 톤
- 1인칭 단수 ("저는 / I" — "우리는 / We" 자제, 1인 사업자 톤)
- 영업 톤 X — 도움 톤 O
- 자조적 유머 OK, 비교 비하 X
- 한국어/영어 어조 일관성

### 금칙어 (한국어)
- "대박", "역대급" — 과장된 인터넷 슬랭
- "지금 당장", "마지막 기회" — false urgency
- "보장합니다", "확실합니다" — over-claim
- "절대", "100%" — 절대 표현

### 금칙어 (English)
- "Best ever", "#1", "The only" — superlatives
- "Guaranteed", "Promised" — over-claim
- "You won't believe", "Mind-blowing" — clickbait
- "Limited time" (실제 한정 없을 때) — urgency lie
- "Earn $X" — income claim

### 권장 표현
| 안 좋음 | 좋음 |
|---|---|
| "최고의 자동화" | "내가 매주 쓰는 자동화" |
| "100% 보장" | "한 달 굴려보고 안 맞으면 환불" |
| "마지막 기회!" | "이번 달 추가 슬롯 5개" |
| "지금 당장 시작" | "오늘 셋업하면 내일부터 작동" |

## 시스템 메시지에 포함되는 형태

`ad-variant-generate.md`의 시스템 메시지 끝부분 (이미 포함됨):

```
Guardrails (skip variants that violate these):
- No superlatives ("best", "#1", "the only")
- No income claims ("earn $X", "guaranteed results")
- No before/after weight or appearance
- No urgency lies ("last chance" when not true)
- No clickbait ("you won't believe", "this one trick")
```

위 룰을 한국어 사용 환경에서는 다음 추가 권장:
- 과장 슬랭 ("대박", "역대급") 금지
- "확실히/보장" 류 over-claim 금지
- 가짜 긴급성 ("지금 당장!") 금지

## 자동 필터링 (v2)

현재 MVP는 LLM이 가드레일을 따르도록 프롬프트에 명시. 위반 시 사람 검수에서 폐기.

v2에서 자동 필터링 추가:
1. variants 생성 후 별도 Code 노드에서 정규식·키워드 매칭
2. 위반 variants는 `status=archived` 또는 제거
3. 위반 빈도 높으면 프롬프트 보강

## 광고 정책 변경 모니터링

Meta와 Google은 광고 정책을 자주 업데이트. 분기별로:
1. https://transparency.fb.com/policies/community-standards/ 변경 확인
2. https://support.google.com/adspolicy/ 변경 확인
3. 본 문서 + ad-variant-generate.md 시스템 프롬프트 동기화

## 검수 체크리스트 (사람용)

월요일 검수 시 각 variant에 대해:

- [ ] 본문에 superlative 없음
- [ ] 결과 약속이 현실적 (구체적 + 조건부)
- [ ] 가짜 긴급성 없음
- [ ] 브랜드 톤과 일치
- [ ] 한국어/영어 일관성
- [ ] 헤드라인 40자 / 본문 90-125자 길이 준수
- [ ] CTA가 우리 브랜드 보이스와 어울림

체크 4개 이하 통과 → `status=archived`. 6개 이상 → `status=approved`로.
