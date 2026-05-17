# 04. 경쟁사 인텔리전스 봇

> 경쟁사 블로그·SNS·광고·가격 페이지 변화 일간 모니터링 → 변경 시에만 Slack 알림

## 1. 페인포인트

- 경쟁사 뭐하는지 수동으로는 절대 못 따라감 (3~5개만 돼도 매일 1시간)
- 광고 라이브러리·가격 변화는 깜빡 잊고 안 봄
- "어제 경쟁사가 이거 했네" 알아도 이미 늦음

## 2. KPI

| 지표 | 목표 |
|---|---|
| 모니터링 경쟁사 수 | 5+ |
| 변경 감지 → 알림 시간 | < 24시간 |
| 알림 → 액션 전환률 | 30%+ |

## 3. Input / Output

**Input**
- 경쟁사 도메인 리스트
- 모니터링 대상 URL/계정:
  - 블로그/뉴스룸 URL (RSS or 본문 diff)
  - 가격 페이지 URL
  - IG·X·LinkedIn 계정
  - Meta Ad Library (경쟁사 페이지 ID)

**Output**
- Slack `#competitor-intel` 채널 알림
- Notion `CompetitorTimeline` DB 적재 (시계열)

## 4. n8n 워크플로

```
[Cron: 매일 오전 8시]
        │
        ▼
[For each 경쟁사]
  ├─▶ Blog RSS / 새 글 감지
  ├─▶ 가격 페이지 HTML fetch → 이전 버전과 diff
  ├─▶ SNS 신규 포스트 (IG/X/LinkedIn)
  ├─▶ Meta Ad Library API → 신규 광고 수·소재
  └─▶ Product Hunt / Hacker News 멘션
        │
        ▼
[Diff/Compare]
  - 이전 스냅샷 vs 현재
  - 변경 사항만 추출
        │
        ▼
[Claude API: 변경 요약 + 함의]
  "이 변경이 우리에게 시사하는 점:
   - 카테고리: 가격/포지셔닝/신기능/콘텐츠 전략
   - 우리 액션: ..."
        │
        ▼
[Filter] 의미 있는 변경만 (오타 수정 같은 건 무시)
        │
        ▼
[Notion: CompetitorTimeline 적재]
        │
        ▼
[Slack 메시지] 경쟁사·변경·함의·액션 제안
```

## 5. 필요한 통합

- RSS reader (n8n 내장)
- HTML scraper (Cheerio/Playwright)
- HTML diff 라이브러리
- Meta Ad Library API
- X/LinkedIn API (또는 스크래퍼)
- Claude API
- Notion API
- Slack

## 6. MVP 범위 (Week 3)

**포함**
- 블로그 RSS + 가격 페이지 diff + Meta Ad Library
- 경쟁사 3개로 시작
- Slack 알림 + Notion 적재

**제외 (v2)**
- SNS 모니터링
- HN/Product Hunt 멘션
- 자동 액션 제안 자동화

## 7. 의존성

- **독립 실행 가능**
- **시너지**: 06(광고 진화)에서 경쟁사 광고 패턴 참고
- **시너지**: 01(아이디어 엔진)에서 경쟁사 인기 글 토픽 시드로 활용

## 8. 리스크

- 스크래핑 차단 (Cloudflare) → Playwright + 프록시 검토
- HTML diff 노이즈 (광고 배너, 타임스탬프) → CSS selector로 핵심 영역만 비교
- 법적 이슈: 공개 정보만, 로그인 영역 절대 금지

## 9. 프롬프트

`prompts/`:
- `competitor-change-analyze.md` (변경 요약 + 함의)
- `competitor-action-suggest.md` (우리 액션 제안)
