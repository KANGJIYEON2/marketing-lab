# 09. 주간 마케팅 리포트 + 다음주 액션 추천

> 일요일 밤 모든 KPI 자동 집계 → 월요일 9시 Slack에 인사이트 + 액션 3개

## 1. 페인포인트

- 매주 월요일에 GA4·광고·이메일·SNS 숫자 모으느라 오전 사라짐
- 숫자만 보는 건 의미 없음 → "다음에 뭐할까"가 진짜 필요
- 다른 자동화들의 실행 결과(01의 글감 수, 05의 SEO 액션 진행률 등)도 한 눈에 보고 싶음

## 2. KPI

| 지표 | 목표 |
|---|---|
| 리포트 생성 시간 (사람) | 0분 (전자동) |
| 액션 추천의 실행률 | 50%+ |
| 주간 KPI 트렌드 추적 채널 | 6+ |

## 3. Input / Output

**Input**
- GA4: 세션·유저·전환·소스별 트래픽
- Meta Ads + Google Ads: 지출·ROAS·CPA·CTR
- 이메일 (Resend/Mailchimp): 발송·오픈·클릭·답장
- SNS: IG/X/LinkedIn 도달·인게이지먼트
- 다른 자동화의 Notion DB:
  - 01: 신규 글감 수
  - 02: 발행 콘텐츠 수
  - 05: SEO 액션 진행률
  - 06: 광고 변형 수, 위너
  - 07: 신규 리드, MQL
  - 10: 진행 중 실험·위너

**Output**
- Slack `#weekly-report` 게시 (월요일 9시)
- Notion `WeeklyReports` 아카이브

## 4. n8n 워크플로

```
[Cron: 매주 일요일 22시]
        │
        ▼
[병렬 데이터 수집]
  ├─▶ GA4 API (이번 주 vs 지난 주)
  ├─▶ Meta Ads API
  ├─▶ Google Ads API
  ├─▶ Resend/Mailchimp API
  ├─▶ Meta Graph (IG insights)
  ├─▶ X/LinkedIn API
  └─▶ Notion (자동화별 DB 카운트)
        │
        ▼
[Aggregate] 공통 스키마로 정규화
        │
        ▼
[Claude API: 인사이트 + 액션]
  "이번 주 데이터:
   - 트래픽: X (vs 지난주 +Y%)
   - 광고: ROAS X
   - 리드: X (MQL Y)
   - 콘텐츠 발행: X
   - ...
   
   인사이트 3개 + 다음주 액션 3개를 제안하라.
   액션은 구체적이고 실행 가능해야 함."
        │
        ▼
[Notion: WeeklyReports 적재]
        │
        ▼
[Schedule: 월요일 9시 Slack 게시]
  - 메인 KPI 표
  - 인사이트 3개
  - 액션 3개 (각각 어떤 자동화 폴더와 연관되는지 링크)
```

## 5. 필요한 통합

- GA4 Data API (서비스 계정)
- Meta Marketing API
- Google Ads API
- Resend / Mailchimp API
- Meta Graph (IG)
- X/LinkedIn API
- Notion API
- Claude API
- Slack scheduled message

## 6. MVP 범위 (Week 6~7)

**포함**
- GA4 + Meta Ads + 이메일 3개만
- KPI 표 + Claude 인사이트
- Slack 게시 (수동 트리거로 시작)

**제외 (v2)**
- 다른 자동화 DB 연동 (해당 자동화가 어느 정도 자리잡은 후)
- 자동 액션 → 자동화 실행 연결 (예: "광고 변형 생성" 액션이 06 자동 실행)
- 이메일/PDF 포맷 리포트 (현재는 Slack only)

## 7. 의존성

- **모든 자동화의 마지막 단계**
- **선행**: 다른 자동화가 어느 정도 데이터 쌓은 후 (6주차 이후 권장)
- 단, GA4·광고만 보는 MVP는 1주차에도 가능

## 8. 리스크

- API 한도 (특히 GA4 Data API quota) → 캐싱 + 일 1회 호출
- Claude 인사이트의 진부함 → 프롬프트에 "지난 4주 트렌드 대비"를 강조
- 액션 추천이 너무 일반적 → 실제 자동화 결과 데이터를 컨텍스트로 풍부하게

## 9. 프롬프트

`prompts/`:
- `weekly-insights-actions.md`
- `kpi-trend-detect.md` (이상치·트렌드 자동 감지)
