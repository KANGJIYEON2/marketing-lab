# weekly-insights-actions

전 자동화의 KPI 스냅샷 → 인사이트 3개 + 액션 3개를 단일 호출로.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.4 (분석 일관성 + 약간의 표현 다양성)
- max tokens: 1500

## 시스템 메시지

```
You are a CMO summarizing a 1-week marketing performance snapshot for a solo founder.

Language: {ko | en}

Return STRICT JSON:
{
  "insights": [
    {"title": "<short title>", "detail": "<one sentence with specific numbers>"},
    ... 3 items
  ],
  "actions": [
    {"title": "<imperative action>", "why": "<one sentence>", "linked_automation": "<e.g., 06-ad-evolution, 05-seo-hunter, or 'none'>"},
    ... 3 items
  ],
  "headline_metric": "<the one number to highlight, with trend>",
  "warning": "<one urgent issue to address, or empty>"
}

Insight rules:
- Each insight references a specific KPI with comparison (vs last week)
- Highlight surprises (large trend changes, anomalies)
- Connect data across sources (e.g., 'leads up but inbox response rate down')

Action rules:
- Imperative form
- Tied to a specific automation when relevant (use folder name like '06-ad-evolution')
- Doable in one week
- Maximum 3 actions — prioritize over completeness

No prose, no markdown, no code fences.
```

## 사용자 메시지 (입력)

```
This week's snapshot:
{
  "week_label": "2026-05-10 ~ 2026-05-17",
  "traffic": { "sessions": 4200, "users": 3100, "conversions": 38, "sessions_trend": "+12%", "conversions_trend": "+25%" },
  "ads": { "spend": 480, "impressions": 85000, "clicks": 1200, "purchases": 14, "ctr": 1.4, "cpc": 0.4, "roas": 2.8 },
  "email": { "sent_count": 156 },
  "leads": { "total": 42, "hot": 8 },
  "content": { "published_count": 3 },
  "inbox": { "total": 28, "replied": 22, "response_rate": "78%" },
  "competitor": { "high_impact_count": 1 },
  "seo": { "open_opportunities": 14, "completed": 2 }
}
```

## 출력 예시

### Case 1 — 좋은 주

```json
{
  "headline_metric": "ROAS 2.8x · 전환 +25% (38건)",
  "warning": "",
  "insights": [
    {
      "title": "광고 효율 회복",
      "detail": "이번 주 ROAS 2.8x로 회복 (지난주 2.1x). CTR 1.4%는 4주 만에 최고."
    },
    {
      "title": "리드 vs 응답률 불균형",
      "detail": "신규 리드 42건 중 hot 8건. Inbox 응답률 78%로 안정적이나 hot 리드 후속 조치 누락 가능성."
    },
    {
      "title": "SEO 작업 적체",
      "detail": "Open opportunities 14건 vs completed 2건. 발굴은 잘 되지만 실행 속도 부족."
    }
  ],
  "actions": [
    {
      "title": "Hot 리드 8건 미팅 일정 잡기",
      "why": "최근 4주 hot 리드 → 미팅 전환률 30%대인데 이번 주 회수 안 되면 다음 주 영향.",
      "linked_automation": "07-lead-sequence"
    },
    {
      "title": "SEO opportunities Top 3 이번 주 안에 글 업데이트",
      "why": "Open 14건 누적 중. priority_score 가장 높은 3개 처리하면 다음 GSC 사이클에서 클릭 +120 예상.",
      "linked_automation": "05-seo-hunter"
    },
    {
      "title": "광고 위닝 패턴으로 신규 변형 2개 추가 업로드",
      "why": "ROAS 회복 시점에 변형 추가하면 동력 유지 가능. 현재 광고세트당 1-2개만 활성.",
      "linked_automation": "06-ad-evolution"
    }
  ]
}
```

### Case 2 — 경고가 있는 주

```json
{
  "headline_metric": "ROAS 1.4x · 4주 최저",
  "warning": "광고 ROAS가 손익분기점 (1.0x) 근접. 즉시 점검 필요.",
  "insights": [
    {
      "title": "ROAS 급락",
      "detail": "이번 주 ROAS 1.4x (지난주 2.5x). 광고 피로도 + 잘 안 되는 변형이 예산 잡아먹는 중."
    },
    {
      "title": "콘텐츠 발행 정체",
      "detail": "이번 주 발행 0편 (지난주 3편). SEO opportunities 18건 누적 — 콘텐츠 백로그 큼."
    },
    {
      "title": "경쟁사 가격 인하 감지",
      "detail": "Competitor A 가격 -15% 인하 (impact 9/10). 우리 LP 전환률 모니터링 필요."
    }
  ],
  "actions": [
    {
      "title": "Bottom 30% 광고 즉시 일시정지",
      "why": "ROAS 1.4x는 손익분기 근접. 단기 조치로 예산 회수 + 변형 진화 사이클 가속.",
      "linked_automation": "06-ad-evolution"
    },
    {
      "title": "유튜브 영상 1편 → 02 자동화로 콘텐츠 5종 양산",
      "why": "발행 백로그 해결. 02는 영상 1편으로 콘텐츠 5~7개 만들어줌.",
      "linked_automation": "02-youtube-factory"
    },
    {
      "title": "LP 가격 차별화 메시지 A/B 실험",
      "why": "경쟁사 가격 인하 대응. 단순 가격 매칭보다 차별화 가치(셀프호스팅 등) 강조 효과 측정.",
      "linked_automation": "10-lp-ab-test"
    }
  ]
}
```

## 평가 기준 (첫 4주 매주 점검)

- [ ] `headline_metric`이 한 줄로 이번 주의 핵심을 전달
- [ ] `insights`가 **데이터 교차**해서 새로운 관점 제시 (단순 숫자 나열 아님)
- [ ] `actions`가 모두 imperative, 1주 안에 실행 가능
- [ ] `linked_automation`이 실제 폴더 이름과 일치 (자동화 연계 정확)
- [ ] `warning`이 진짜 긴급할 때만 채워짐 (남발 X)

## 좋은 인사이트의 조건

❌ 나쁜 인사이트: "이번 주 세션 4200개로 증가했습니다."
→ 그냥 숫자 보고. AI 가치 없음.

✅ 좋은 인사이트: "세션 +12%지만 전환률 0.9%로 정체. 트래픽 품질 보다는 LP 전환 측이 병목."

차이: 두 KPI 교차 + 진단 + 함의.

## 자주 발생하는 LLM 실수

1. **숫자 단순 반복** — insights가 그냥 표 재서술
2. **5개 이상 액션** — 우선순위 약화. 3개로 강제
3. **`linked_automation` 추측** — 존재하지 않는 폴더 이름 작성. 다음 9개 중 하나여야:
   - `01-idea-engine`, `02-youtube-factory`, `03-unified-inbox`, `04-competitor-intel`, `05-seo-hunter`, `06-ad-evolution`, `07-lead-sequence`, `08-ugc-collector`, `10-lp-ab-test`
4. **warning 남발** — 매주 warning이 있으면 신호 가치 ↓. 진짜 긴급할 때만

## 데이터 누락 시

일부 source가 실패하거나 비어있어도 (continueOnFail) LLM은 사용 가능한 것만으로 분석. 누락된 영역은 인사이트/액션에서 제외.

## 비용

- 입력: ~1.5k tokens (snapshot + system)
- 출력: ~800 tokens
- 주 1회 호출 = 월 ≈ $0.10
