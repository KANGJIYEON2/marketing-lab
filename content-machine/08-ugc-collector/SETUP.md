# 08 UGC 컬렉터 — 세팅 가이드

`workflows/ugc-collector.json`을 임포트해서 매일 9시에 IG 태그·해시태그 자동 수집.

**선행**: [Layer 0 SETUP](../../docs/layer-0/SETUP.md) + `UGCLibrary` DB

**소요 시간**: 30분~1시간 (Meta 권한 03·04에서 셋업 됐다면)

---

## 0. 준비물 체크

- [ ] Layer 0 + [UGCLibrary DB](../../docs/notion-schemas/ugc-library.md)
- [ ] **Instagram Business 계정** (03에서 셋업)
- [ ] `META_GRAPH_ACCESS_TOKEN` + IG 권한
- [ ] 브랜드 해시태그 1~3개 (제품명·브랜드명)

---

## 1. Meta IG 권한 확장

03 통합 인박스에서 발급한 토큰을 그대로 쓰되, 다음 권한이 포함되어야:

| 권한 | 용도 |
|---|---|
| `instagram_basic` | 기본 |
| `instagram_manage_comments` | 03에서 사용 |
| `pages_show_list` | 03에서 사용 |
| `pages_read_engagement` | 03에서 사용 |
| **`instagram_content_publish`** | 본 자동화는 안 쓰나 v2 자동 리포스트용 |

`/tags` endpoint와 `/ig_hashtag_search`는 위 권한으로 작동.

### 토큰 권한 확인

```bash
curl "https://graph.facebook.com/v19.0/me/permissions?access_token={TOKEN}" | jq
```

부족하면 Graph API Explorer에서 재발급.

---

## 2. 브랜드 해시태그 설정

`.env`:
```bash
BRAND_HASHTAGS=yourbrand,yourproduct
BRAND_MENTIONS=yourbrand
UGC_LOOKBACK_HOURS=24
```

### 해시태그 선택 팁

- **브랜드명** (`#yourbrand`) — 가장 많은 UGC
- **제품명** (`#yourproduct`) — 더 구체적
- **캠페인 해시태그** (`#yourbrand2026`) — 시기별

### 검색 한도

Meta `/ig_hashtag_search`:
- **24시간 내 30개 해시태그**까지만 검색 가능 (Meta 정책)
- 본 워크플로는 매일 1회 × 해시태그 수만큼 호출 → 2-3개 권장

너무 많이 검색하면 4xx 에러. 핵심 해시태그만.

---

## 3. IG 태그 수집 (별도 권한 불필요)

`/{ig-user-id}/tags` endpoint는 "내 IG 계정이 사진 속에 태그된" 게시물 반환. `instagram_manage_comments` 권한으로 작동.

브랜드 멘션(@username)은 직접적 API가 없지만, 태그가 거의 동일한 신호. 멘션은 03 통합 인박스의 IG 댓글 분기에서도 잡힘 (멘션이 댓글 형태로 도착).

---

## 4. Notion UGCLibrary DB

[`UGCLibrary` 스키마](../../docs/notion-schemas/ugc-library.md) 모든 속성 + Select 옵션 사전 정의:
- `channel`: `instagram_mention`, `instagram_tag`, `instagram_hashtag` 미리 추가
- `sentiment`: 3종 모두
- `usage_formats` (Multi-select): 5종 모두
- `permission_status`: `needs_permission` 등 5종

Integration 연결 + `.env`:
```bash
NOTION_DB_UGC_LIBRARY=<32자 hex>
```

---

## 5. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/ugc-collector.json`

### 임포트 후 확인

- [ ] **IG: Tagged Media / Resolve Hashtag ID / Hashtag: Recent Media**: env var 참조 (META_IG_USER_ID, META_GRAPH_ACCESS_TOKEN)
- [ ] **Already In Library? / Notion: Add to Library**: UGCLibrary DB
- [ ] **Claude: UGC Classify**: Anthropic credential
- [ ] **Slack: UGC Digest**: channel `$env.SLACK_CHANNEL_LOG`
- [ ] **Config** 노드 — `min_usability_score` 7 (필터 임계값)

---

## 6. 첫 실행 (수동 테스트)

`Daily 09:00` 트리거 → **Execute Workflow**.

### 통과 시그널

| 노드 | 확인 |
|---|---|
| IG: Tagged Media | 최근 태그 게시물 (없으면 빈 결과 — 정상) |
| Resolve Hashtag ID | 각 해시태그의 ID |
| Hashtag: Recent Media | 해시태그 게시물 |
| Merge Sources | 통합된 UGC 배열 |
| Claude: UGC Classify | sentiment·score·formats JSON |
| Quality + Positive | score≥7 AND positive만 통과 |
| Notion: Add to Library | DB 행 생성 (`permission_status=needs_permission`) |
| Slack: UGC Digest | Top 3 알림 (UGC 있을 때만) |

### 데이터 부족 시

- 브랜드 인지도가 낮으면 매일 0건 수집되는 정상
- 일주일 운영하며 누적되는 형태
- 멘션을 받으려면 03 통합 인박스도 함께 운영 권장

---

## 7. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| `ig_hashtag_search` 빈 결과 | 해시태그가 IG에 없거나 검색 한도 초과 | 다른 해시태그 시도, 24시간 후 재시도 |
| `/tags` 401 | 권한 누락 | 1번 단계 토큰 재발급 |
| Notion `multi_select` 형식 오류 | `usage_formats` 옵션 미생성 | 5개 옵션 모두 DB에 미리 추가 |
| Slack 메시지 텅 빔 | 임계값 미달 또는 데이터 0 | 정상. 어떤 날은 수집 0건일 수 있음 |
| Claude `usability_score` 항상 9-10 | 평가 기준 너무 관대 | 첫 1주 매일 점수 분포 점검 + 프롬프트 보강 |
| 같은 게시물 중복 적재 | `external_id` 미저장 | DB 속성 이름 확인 |

---

## 8. 활성화

수동 테스트 후 토글 **Active**. 매일 9시 자동.

---

## 9. 운영 흐름

### 일간 흐름

1. 매일 9시 Slack에 Top 3 UGC 알림 (수집된 게 있으면)
2. Notion `UGCLibrary` → filter `permission_status = needs_permission` 진입
3. 우선순위 (score 높은 순) 보고 허락 요청 보낼 대상 선택
4. 작성자에게 DM/댓글로 표준 메시지 발송:
   > 안녕하세요! 후기 감사합니다. 저희 채널/광고에 인용해도 될까요? 출처는 명시하겠습니다.
5. 답변 받으면 `permission_status` 수동 업데이트 (`approved` / `denied`)

### 자산 활용

`permission_status = approved` 행을:
- **광고 카피 시드**: 06이 자동으로 `pull_quote` 활용 (v2)
- **LP 후기 섹션**: 사람이 수동으로 LP에 삽입 (10 v2에서 자동)
- **IG 캐러셀**: Bannerbear/Canva API로 텍스트 오버레이 생성 (v2)

### 부정 UGC 처리

`sentiment=negative`로 분류된 행은 자동으로 **저장 안 됨** (Quality + Positive 필터에서 차단). 부정 콘텐츠는 03 통합 인박스에서 별도 처리.

### 비용 추정

| 항목 | 일 30건 기준 |
|---|---|
| Meta API | 무료 |
| Claude UGC Classify | ≈ $0.15/일 = 월 $4.5 |
| **합계** | **약 $4.5/월** |

---

## 10. v2 확장 후보

PRD `MVP 범위 (제외)` 항목:

1. **자동 가공 (캐러셀·광고 소재)** — Bannerbear API로 텍스트 오버레이, 브랜드 컬러 적용
2. **자동 LP 후기 위젯** — `permission_status=approved` 추가 시 LP의 testimonial 섹션 자동 업데이트
3. **자동 허락 요청 DM** — Meta Graph API로 표준 메시지 발송 + 답변 감지
4. **X·네이버 블로그·리뷰 사이트 채널 추가** — 별도 분기 추가
5. **위·허위 후기 필터링** — 계정 신뢰도 점수 (팔로워·게시물 수)

각각 별도 PR로.

---

## 11. 다음 단계

- 첫 1주 수집량 측정 — 0건이면 인지도/해시태그 재검토
- 자산화율(approved 비율) 추적 — 허락 요청 응답률
- 다음 자동화 — 본 자동화로 content-machine 카테고리 완성. 남은 건 [10 LP A/B](../../revenue-impact/10-lp-ab-test/)와 [09 주간 리포트](../../ops-reduction/09-weekly-report/)
