# daily-digest

Slack `#content-ideas` 채널 일간 다이제스트 포맷.

이 작업은 LLM 호출이 아니라 **결정론적 템플릿**이다. 워크플로의 `Build Slack Digest` Code 노드에 인라인 구현되어 있다. 이 문서는 포맷 명세 + 향후 LLM 강화 시 참고용.

---

## 현재 (결정론적) 포맷

```
🎯 오늘의 콘텐츠 아이디어 {N}개
신규 수집 (score >= {min_score})

1. {title}
   페인: {pain_point}
   키워드: `{keyword}` · score {N} · {source}
   <{source_url}|소스 보기>

2. ...

전체 보기: Notion ContentIdeas DB
```

- 상위 5개만 표시 (score 내림차순)
- 6개째부터는 카운트만 ("외 N개")
- Slack mrkdwn 문법 사용

## v2 (LLM 강화) — 검토 후 적용

매일 아침 단순 리스트 대신, Claude가 **테마 군집화 + 주간 트렌드 코멘트** 추가:

```
오늘의 콘텐츠 아이디어 N개 (이번 주 누적 M개)

🔥 떠오르는 테마: "AI 에이전트 운영 부담"
관련 글감 3건이 모두 동일 페인 — 가장 시급한 토픽일 가능성.

📝 추천 글감 Top 5
1. ...

📊 이번 주 키워드 분포
- n8n automation: 12건
- saas growth: 8건
- ...
```

### v2 시스템 프롬프트 (초안)

```
You are reviewing the day's collected content ideas.
Identify:
1. Theme clusters: which 2-3 pain points repeat? Group ideas under each.
2. Trending keyword: which keyword saw the biggest jump vs last 7-day avg?
3. Top 5 picks: rank by combined score (LLM score × engagement × theme cluster size).

Return formatted Slack mrkdwn message, max 800 chars.
```

### 트리거 변경

v1: 자동화 끝부분에서 자동 발송
v2: 별도 cron으로 분리 (매일 09:00) — Notion DB를 입력으로 사용 → LLM 클러스터링 → Slack

## 액션 버튼 (v2)

각 글감 옆에 Slack 인터랙티브 버튼:
- ✍️ 초안 만들기 → 02 워크플로 또는 별도 글 생성 워크플로 트리거
- 📌 즐겨찾기 → Notion status `queued`로
- ❌ 무시 → Notion status `archived`
