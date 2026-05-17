# kpi-trend-detect (v2)

⚠️ **MVP에 포함되지 않음.** MVP는 이번 주 vs 지난 주 단순 비교만. 4주 트렌드·이상치 감지는 v2.

## v2가 필요한 이유

이번 주 vs 지난 주만 보면:
- 일시적 fluctuation을 트렌드로 오인 (false positive)
- 점진적 악화를 놓침 (true negative)
- 계절성·요일 패턴 무시

## v2 구현 방식

별도 LLM 호출 또는 단일 호출에 통합:

### Option A: 통합 (시스템 메시지 확장)

`weekly-insights-actions.md`의 시스템 메시지에 추가:

```
You are also given 4 weeks of weekly snapshots. For each KPI:
- Identify the trend (improving / stable / declining / volatile)
- Flag outliers (>2 standard deviations from 4-week mean)
- Distinguish 'this week happens to be lower' vs 'declining trend over 3+ weeks'

Insights should reference trends, not just point comparisons.
```

입력에 4주 데이터 array 추가:
```json
{
  "this_week": {...},
  "previous_4_weeks": [
    {...week1...},
    {...week2...},
    {...week3...},
    {...week4...}
  ]
}
```

### Option B: 분리 (전처리 호출)

1. 첫 호출: 4주 데이터 → 트렌드 분류 (각 KPI별로 improving/stable/declining/volatile)
2. 둘째 호출: 트렌드 + 이번 주 → 인사이트·액션

장점: 더 정확. 단점: 호출 2배.

MVP 검증 후 결정.

## 통계적 검출 (코드 기반)

`Build KPI Snapshot` Code 노드를 확장해 4주 평균·표준편차 계산:

```js
function trend(values) {
  // values = [week-4, week-3, week-2, week-1]
  if (values.length < 3) return 'insufficient_data';
  const n = values.length;
  const mean = values.reduce((a, b) => a + b, 0) / n;
  const sd = Math.sqrt(values.reduce((sum, x) => sum + (x - mean) ** 2, 0) / n);
  const latest = values[values.length - 1];

  // outlier check
  if (Math.abs(latest - mean) > 2 * sd) return 'outlier';

  // trend check (linear regression slope sign)
  const slope = computeSlope(values);
  if (Math.abs(slope) < sd * 0.1) return 'stable';
  return slope > 0 ? 'improving' : 'declining';
}
```

이 트렌드 분류를 LLM에 함께 전달하면 더 정확한 인사이트.

## 추가 source 필요 (v2)

매주 적재된 weekly_report 자체를 CampaignMetrics에서 fetch — 본 MVP에서 archive 행 만들고 있음 (`Notion: Archive Report` 노드). 4주 전 행까지 가져와서 트렌드 계산.

쿼리: `entity_type=campaign AND entity_id startsWith 'weekly-report-'`, sort `date desc`, limit 4.

## 이상치(anomaly) 우선순위

자동화가 이상치를 감지하면 `warning` 필드에 자동 게시:

| 이상치 | 우선순위 | 워닝 메시지 예시 |
|---|---|---|
| ROAS > 2σ 하락 | high | "ROAS가 4주 평균 대비 ~50% 하락 — 즉시 광고 검토" |
| Inbox response rate > 2σ 하락 | high | "응답률 4주 최저 — 답변 누락 확인" |
| Conversions > 2σ 상승 | medium | "이번 주 전환 폭증 — 무엇이 변했나? 패턴 분석 권장" |
| Sessions = 0 | critical | "트래픽 0 — 사이트 다운 가능성" |

## 구현 시점

본 자동화 운영 4주 이상 후. 그 전에는 데이터 부족.

## 통계 한계

- 4주는 통계적으로 매우 짧음. 신뢰도 ↓
- 이상치 판단은 사람 검수 필수
- 계절성(분기/연간 패턴) 잡으려면 1년 이상 데이터 필요

이런 한계 인지하고 운영해야 함. 자동화는 의사결정의 보조이지 대체가 아님.
