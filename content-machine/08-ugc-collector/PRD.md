# 08. UGC·리뷰 컬렉터

> 외부에서 만들어진 리뷰·태그·멘션을 **광고·LP·SNS 소재**로 자동 가공

## 1. 페인포인트

- UGC는 광고보다 전환률 2~3배 높지만 수집·정리가 번거로움
- 좋은 후기가 있어도 어디서 봤는지 잊어버림
- 캐러셀·광고 소재로 가공하는 데 매번 30분~1시간

## 2. KPI

| 지표 | 목표 |
|---|---|
| 월간 수집 UGC | 30개+ |
| UGC → 활용 콘텐츠 | 50%+ (캐러셀/광고/LP) |
| UGC 광고의 ROAS 가산 | 베이스 대비 +20%+ |

## 3. Input / Output

**Input**
- 모니터링 대상: 브랜드명, 제품명, 해시태그, 멘션 (@)
- 채널: Instagram, X, Threads, 네이버 블로그, 리뷰 사이트

**Output**
- Notion `UGCLibrary` DB
  - `source_url`, `author`, `content`, `sentiment`, `permission_status`, `usage_format[]`
- Google Drive: 자동 가공된 캐러셀 이미지 / 광고 소재

## 4. n8n 워크플로

```
[Cron: 매일 1회]
        │
        ▼
[병렬 수집]
  ├─▶ Meta Graph API (IG 멘션·태그)
  ├─▶ X Search API (브랜드 키워드)
  ├─▶ 네이버 블로그 검색 API
  └─▶ 리뷰 사이트 (Trustpilot, 네이버 플레이스 등)
        │
        ▼
[Dedup] URL/content 해시
        │
        ▼
[Claude API: 분석]
  - 감성 (긍정/중립/부정)
  - 활용 가능 포맷 (캐러셀/광고/LP 인용/SNS 리포스트)
  - 추출 가능한 핵심 인용구
        │
        ▼
[Filter] 긍정 only, score >= 7
        │
        ▼
[Notion: UGCLibrary 적재]
  status = needs_permission
        │
        ▼
[Slack] "신규 UGC 5개 — 사용 허락 받을 우선순위:"
        │
        ▼
[사람: 허락 요청 메시지 발송]
status = approved
        │
        ▼
[자동 가공 트리거]
  ├─▶ 캐러셀 이미지 (Bannerbear 또는 Canva API)
  ├─▶ 광고 소재 (LP 인용 + 이미지 합성)
  └─▶ LP 후기 섹션 (HTML 스니펫)
```

## 5. 필요한 통합

- Meta Graph API (Business 계정)
- X API
- 네이버 검색 API
- Claude API
- Notion API
- Bannerbear 또는 Canva API (이미지 자동 생성)
- Google Drive API

## 6. MVP 범위 (Week 4~5)

**포함**
- IG 멘션·태그 수집
- Claude 분석 + Notion 적재
- 허락 요청은 수동, 가공은 v2

**제외 (v2)**
- 자동 캐러셀 생성
- 광고 소재 자동 합성
- LP 후기 위젯 자동 업데이트

## 7. 의존성

- **선행**: 브랜드/제품이 멘션될 만큼 인지도 필요 (없으면 06·07 먼저)
- **시너지**: 06(광고 진화)이 UGC 라이브러리에서 시드 가져옴

## 8. 리스크

- 사용 권한: 허락 없이 사용 금지. 워크플로에 명시적 단계 포함
- 부정 리뷰 처리: 별도 알림으로 운영팀 라우팅 (자산화는 긍정만)
- 위·허위 후기 필터링: 계정 신뢰도 점수 도입 검토

## 9. 프롬프트

`prompts/ugc-classify.md`
- input: 멘션 raw content
- output: JSON `{sentiment, formats[], pull_quote, usability_score}`
