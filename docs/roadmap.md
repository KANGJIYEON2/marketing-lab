# 12주 로드맵

10개 마케팅 자동화를 **MVP 적층** 방식으로 12주에 걸쳐 구축. 각 주차는 1~2개 자동화의 MVP 완성을 목표로 한다.

## 핵심 원칙

1. **첫 주는 인프라만** — Notion DB·n8n·Slack 연결. 자동화 0개.
2. **콘텐츠 먼저** — 다른 레이어의 input이 콘텐츠. 1·2번이 잘 돌면 6·10번이 쉬워짐.
3. **MVP 후 즉시 실사용** — 다음 자동화 시작 전에 직전 MVP를 실제 일주일 굴려본다. 안 굴리는 자동화는 만들지 말 것.
4. **병렬 작업 최소화** — 한 번에 1~2개만. 동시에 5개 시작하면 0개 완성됨.

---

## 주차별 일정

### Week 1 · Layer 0: 인프라

목표: **자동화 0개 만들고도 일주일을 끝낼 수 있어야 한다.**

- [ ] n8n 셀프호스팅 셋업 (Docker/Railway/Hetzner)
- [ ] Notion 워크스페이스 + DB 4종 생성
  - `ContentIdeas`, `ContentDrafts`, `Leads`, `CampaignMetrics`
- [ ] Slack 워크스페이스 + 채널 생성
  - `#automation-log`, `#content-ideas`, `#inbox`, `#competitor-intel`, `#weekly-report`
- [ ] n8n credentials 등록
  - Claude API, OpenAI, Notion, Slack
- [ ] `.env` 템플릿 + `.gitignore` 정리

산출물: 빈 n8n에서 "hello world" 워크플로 1개가 Slack에 메시지 보낸다.

---

### Week 2 · 01 콘텐츠 아이디어 엔진 (MVP)

| 카테고리 | content-machine |
|---|---|
| 의존 | Layer 0 |

**할 일**
- [ ] Reddit + X 키워드 검색 워크플로
- [ ] Claude로 페인포인트 추출 + 점수
- [ ] Notion `ContentIdeas` 적재
- [ ] Slack 일간 다이제스트

**완료 기준**: 매일 아침 7시에 글감 5~10개가 Notion에 자동 추가된다.

---

### Week 3 · 02 유튜브 팩토리 (MVP) + 03 통합 인박스 (MVP)

| 02 | content-machine |
|---|---|
| 03 | ops-reduction |

**02 할 일**
- [ ] YouTube RSS 신규 영상 감지
- [ ] 자막 추출 + Claude로 블로그 + X 스레드 + LinkedIn 생성
- [ ] Notion `ContentDrafts` 적재

**03 할 일**
- [ ] IG + 이메일만 수집 → Slack `#inbox`
- [ ] 분류 + 답변 초안

**완료 기준**:
- 02: 영상 업로드 후 1시간 내 파생 콘텐츠 3종 Notion에 도착
- 03: 모든 IG/이메일 문의가 Slack에 답변 초안과 함께 도착

---

### Week 4 · 04 경쟁사 인텔 (MVP) + 07 리드 시퀀스 (MVP)

| 04 | ops-reduction |
|---|---|
| 07 | revenue-impact |

**04 할 일**
- [ ] 경쟁사 3개 등록
- [ ] 블로그 RSS + 가격 페이지 diff + Meta Ad Library
- [ ] Claude 요약 → Slack

**07 할 일**
- [ ] 폼 webhook → Apollo enrich
- [ ] Day 0 환영 메일 자동 발송 (Resend)
- [ ] Notion `Leads` 적재 + Slack 핫리드 알림

**완료 기준**:
- 04: 매일 8시에 경쟁사 변화 알림 (변화 없으면 무알림)
- 07: 폼 제출 5분 내 개인화 환영 메일 발송

---

### Week 5 · 05 SEO 헌터 (MVP) + 06 광고 진화 (MVP)

| 05 | content-machine |
|---|---|
| 06 | revenue-impact |

**05 할 일**
- [ ] GSC API 연동
- [ ] 11~20위 키워드 추출
- [ ] Claude로 액션 아이템 생성
- [ ] Notion `SEOOpportunities` + Slack 주간 다이제스트

**06 할 일**
- [ ] Meta Ads 직전 7일 성과 fetch
- [ ] Top 20% 광고에서 패턴 추출
- [ ] Claude로 카피 변형 10개 생성
- [ ] Notion `AdVariants` 적재 + Slack 검토 알림 (수동 업로드)

**완료 기준**:
- 05: 매주 월요일 SEO 기회 Top 5가 Slack에 도착
- 06: 매주 일요일 광고 카피 변형 10개가 Notion에 준비됨

---

### Week 6 · 09 주간 리포트 (MVP)

| 카테고리 | ops-reduction |
|---|---|

**할 일**
- [ ] GA4 + Meta Ads + Resend 3개 소스만
- [ ] Claude로 인사이트 + 액션 3개
- [ ] 월요일 9시 Slack 자동 게시

**완료 기준**: 월요일 9시 출근 시 이미 Slack에 리포트가 와 있다.

---

### Week 7 · 08 UGC 컬렉터 (MVP) + 10 LP A/B (MVP)

| 08 | content-machine |
|---|---|
| 10 | revenue-impact |

**08 할 일**
- [ ] IG 멘션·태그 수집
- [ ] 감성 분석 + Notion 적재
- [ ] (가공·발행은 v2)

**10 할 일**
- [ ] LP 1개 선택, 헤드라인 A/B
- [ ] Claude로 변형 2개 생성
- [ ] GA4 experiment_id 트래킹
- [ ] 유의성 도달 시 Slack 알림

**완료 기준**:
- 08: 매일 IG에서 브랜드 멘션 자동 수집
- 10: 첫 LP 실험 진행 중 (위너 결정 자동 감지)

---

### Week 8~12 · v2 기능 + 안정화

각 자동화의 v2 기능 (PRD 6번 섹션 "제외" 항목) 단계적 추가.

**우선순위 (영향도 기준)**
1. **07 시퀀스 → 3단계 전체** (Day 2, Day 5 추가) — Week 8
2. **06 광고 → 자동 이미지 생성 + 자동 업로드** — Week 9
3. **02 팩토리 → 자동 발행** (블로그·X·LinkedIn) — Week 10
4. **03 인박스 → Slack 1-click 발송** — Week 10
5. **10 LP A/B → 자동 위너 채택** — Week 11
6. **09 리포트 → 다른 자동화 DB 통합** — Week 11
7. **나머지 v2** — Week 12

---

## 의존성 그래프

```
Week 1: Layer 0
   │
   ├─▶ Week 2: [01 아이디어 엔진]
   │      │
   │      ▼
   ├─▶ Week 3: [02 유튜브 팩토리]   [03 통합 인박스]
   │      │
   │      ▼
   ├─▶ Week 4: [04 경쟁사 인텔]     [07 리드 시퀀스]
   │      │
   │      ▼
   ├─▶ Week 5: [05 SEO 헌터]        [06 광고 진화]
   │      │                              ▲
   │      │                              │ (02 콘텐츠 활용)
   │      ▼
   ├─▶ Week 6: [09 주간 리포트] ◀──── 모든 자동화 KPI
   │      │
   │      ▼
   └─▶ Week 7: [08 UGC 컬렉터]      [10 LP A/B]
```

---

## 체크인 규칙

매주 일요일 저녁:
1. 직전 주차 MVP 실제로 굴렀는지 확인 (안 굴린 자동화는 폐기 검토)
2. 발견한 이슈는 해당 폴더 `ISSUES.md`에 기록
3. 다음 주차 PRD를 다시 읽고 범위 조정

12주 끝나면:
- 10개 자동화 모두 MVP 이상으로 작동
- 주당 절약 시간 측정 (목표: 15~20시간/주)
- v3 로드맵 작성 (자동화 간 상호 호출 + AI 에이전트화)
