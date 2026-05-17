# 09 주간 리포트 — 세팅 가이드

`workflows/weekly-report.json`을 임포트해서 매주 월요일 9시에 모든 자동화 결과 통합 → Slack 인사이트 + 액션.

**선행**: 다른 자동화가 어느 정도 데이터 쌓은 후 권장 (최소 2~3주)

**소요 시간**: 1~2시간

---

## 0. 준비물 체크

- [ ] [Layer 0](../../docs/layer-0/SETUP.md) 완료 — DB 8종 모두 생성
- [ ] **GA4 + Meta Ads + Resend** credentials (다른 자동화 셋업 시 마련)
- [ ] 다른 자동화 최소 2~3주 운영 (데이터 누적 필요)

---

## 1. 의존 자동화 확인

09는 **다른 모든 자동화의 출력을 통합**. 각 source가 비어있어도 동작하지만 (continueOnFail), 의미 있는 인사이트를 위해:

| 자동화 | 09에 기여하는 데이터 | 운영 필수도 |
|---|---|---|
| [02 유튜브 팩토리](../../content-machine/02-youtube-factory/) | 발행 콘텐츠 수 | 권장 |
| [03 통합 인박스](../03-unified-inbox/) | inbox 메시지·응답률 | 권장 |
| [04 경쟁사 인텔](../04-competitor-intel/) | high-impact 변경 수 | 선택 |
| [05 SEO 헌터](../../content-machine/05-seo-hunter/) | open opportunities·완료 | 권장 |
| [06 광고 진화](../../revenue-impact/06-ad-evolution/) | CampaignMetrics (광고 성과) | 필수 (Meta 광고 운영 시) |
| [07 리드 시퀀스](../../revenue-impact/07-lead-sequence/) | leads·hot leads | 필수 |
| [08 UGC 컬렉터](../../content-machine/08-ugc-collector/) | (v2에서 추가 활용) | 선택 |
| [10 LP A/B](../../revenue-impact/10-lp-ab-test/) | 활성 실험·위너 (v2에서 통합) | 선택 |

---

## 2. 외부 API 권한 통합 확인

다른 자동화에서 이미 발급한 credential 그대로 재사용:

| API | 어디서 발급 | 필요 권한 |
|---|---|---|
| GA4 Data API | 05·10에서 등록한 Google API credential | `analytics.readonly` |
| Meta Marketing API | 03·04·06에서 발급한 `META_GRAPH_ACCESS_TOKEN` | `ads_read` |
| Resend | 07에서 발급한 `RESEND_API_KEY` | List access |
| Notion | Layer 0의 Integration | 모든 DB 연결 |
| Anthropic | Layer 0 | - |
| Slack | Layer 0 | `chat:write` |

신규 발급 불필요. `.env` 그대로.

---

## 3. 워크플로 임포트

n8n → `...` → **Import from File** → `workflows/weekly-report.json`

### 임포트 후 확인

- [ ] **GA4: Traffic / Meta Ads: Week / Resend: Recent**: 각 노드의 인증·env var
- [ ] **Notion 노드 5개** (Leads / Published / Inbox / Competitor / SEO): 각 DB env var
- [ ] **Claude: Insights + Actions**: Anthropic credential
- [ ] **Slack: Weekly Report**: channel `$env.SLACK_CHANNEL_REPORT`
- [ ] **Notion: Archive Report**: CampaignMetrics DB (리포트 자체도 trend 분석용 적재)
- [ ] **Config** 노드 — `report_lang`: `ko` 또는 `en`

---

## 4. 첫 실행 (수동 테스트)

`Monday 09:00` 트리거 → **Execute Workflow**.

### 통과 시그널

| 노드 | 확인 |
|---|---|
| Build Date Ranges | this/prev week ISO 날짜 |
| GA4: Traffic | 2개 dateRange (this, prev) 결과 |
| Meta Ads: Week | 광고 데이터 (운영 중일 때만) |
| Resend: Recent | 메일 array (운영 중일 때만) |
| Notion: 5개 노드 | 각 DB의 past_week 행 |
| Build KPI Snapshot | 종합 객체 (모든 KPI 통합) |
| Claude: Insights + Actions | JSON `{insights, actions, headline_metric, warning}` |
| Slack: Weekly Report | `#weekly-report` 채널에 mrkdwn 메시지 |
| Notion: Archive Report | CampaignMetrics에 weekly row 적재 |

### 데이터 없을 때

첫 운영에서는 대부분 KPI가 0 또는 빈 상태. Claude가 "데이터 부족" 인사이트를 생성하거나 empty insights를 반환할 수 있음. 정상.

운영 2~3주 후부터 의미 있는 분석 가능.

---

## 5. 흔한 에러

| 증상 | 원인 | 해결 |
|---|---|---|
| GA4 dateRange 응답 비어있음 | property에 트래픽 없음 또는 권한 문제 | GA4 Realtime 확인 |
| Meta Ads 401 | 다른 자동화는 작동하는데 09만 실패 | 동일 토큰이라 정상이어야 함. n8n credential 재확인 |
| Notion `past_week` 필터 오류 | n8n Notion 노드 버전 mismatch | 노드 버전 2.2 이상 확인 |
| Claude insights가 빈 배열 | 데이터 부족 또는 LLM 한계 | 운영 2~3주 후 재시도. 또는 시스템 프롬프트의 "if no data, return empty insights" 명시화 |
| Slack 메시지가 너무 길어 잘림 | Slack 4000자 한도 | Build Slack Message Code 노드에서 detail 길이 제한 |
| Notion archive 행 생성 실패 | CampaignMetrics 속성 매핑 오류 | 스키마 확인 |

---

## 6. 활성화

수동 테스트 통과 후 토글 **Active**. 매주 월요일 9시 자동.

월요일 9시 출근 시 이미 Slack에 리포트가 와 있는 상태.

---

## 7. 운영 흐름

### 매주 월요일 9시

1. Slack `#weekly-report` 채널 확인
2. **Headline metric** 한 줄로 이번 주 핵심 파악
3. **Warning**이 있으면 우선 처리
4. **Insights 3개** 읽고 데이터 검증
5. **Actions 3개** — 그 주에 실행할 To-Do
6. 각 action의 `linked_automation`을 통해 해당 폴더로 이동, 작업

### Action 실행 추적

MVP에서는 사람이 수동으로 추적. 권장:
- Slack 메시지에 ✅ reaction 으로 액션 시작 표시
- 완료 시 thread에 결과 댓글

v2에서:
- Action을 별도 Notion `WeeklyActions` DB에 자동 적재 → 완료율 추적
- 미완료 액션은 다음 주 리포트에 carry over

### Slack 메시지 너무 길면

이번 주 출력이 4000자 초과:
- `Build Slack Message` Code 노드에서 각 섹션 줄여 (insights detail slice, ads 섹션 조건부)
- 또는 메인 메시지 + Notion 아카이브 링크 형태로

### 첫 4주 점검 루틴

매주 월요일 출근 후:
- [ ] Insights 3개가 데이터 정확히 반영하는가 (LLM hallucination 점검)
- [ ] Actions 3개 중 실제 실행한 비율 측정
- [ ] linked_automation 정확도 (잘못된 폴더 이름 안 나오는지)

체크 통과율 80% 미달 → `prompts/weekly-insights-actions.md` 보강.

### 다국어

`Config.report_lang`로 ko/en 전환. 데이터는 영어 (GA4·Meta·Resend), 분석만 한국어/영어.

### v2 — 트렌드 감지

[`prompts/kpi-trend-detect.md`](./prompts/kpi-trend-detect.md) 참고. 4주 운영 후 본격 검토.

### 비용 추정

| 항목 | 비용 |
|---|---|
| GA4 / Meta / Resend / Notion API | 무료 (한도 내) |
| Claude (주 1회 인사이트 생성) | $0.10/월 |
| **합계** | **거의 0** |

---

## 8. 마무리

09는 **마지막 자동화**. 본 자동화를 셋업하면 marketing-lab의 10종 풀세트가 작동:

```
Layer 0 인프라
   │
   ├─ 01 아이디어 엔진 → 콘텐츠 시드 공급
   ├─ 02 유튜브 팩토리 → 멀티채널 콘텐츠
   ├─ 03 통합 인박스 → 응대 자동화
   ├─ 04 경쟁사 인텔 → 시장 정보
   ├─ 05 SEO 헌터 → 트래픽 기회
   ├─ 06 광고 진화 → 광고 효율
   ├─ 07 리드 시퀀스 → 리드 자동화
   ├─ 08 UGC 컬렉터 → 자산 라이브러리
   └─ 10 LP A/B → 전환 최적화
        │
        ▼
   09 주간 리포트 ← 모든 결과 통합
```

매주 월요일 9시에 09가 알아서 정리해주는 상태. PM의 시간을 매주 3~5시간 회수.

**다음 단계**: 다른 자동화들의 v2 기능 단계적 추가, 또는 자동화 간 cross-trigger (예: 09 액션이 자동으로 다른 자동화 트리거).

[`docs/roadmap.md`](../../docs/roadmap.md) Week 8~12 참고.
