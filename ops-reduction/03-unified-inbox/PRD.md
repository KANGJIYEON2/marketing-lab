# 03. 댓글·DM·문의 통합 인박스

> 모든 채널 댓글·DM·문의를 한 곳으로 → AI 분류·답변 초안 → 1-click 발송

## 1. 페인포인트

- IG·유튜브·블로그·이메일·X DM 다 따로 봐야 함 → 누락 발생
- 답변 늦으면 매출 손실 (응답 시간 = 전환률)
- 단순 질문도 매번 처음부터 답변 작성

## 2. KPI

| 지표 | 목표 |
|---|---|
| 평균 응답 시간 | < 30분 |
| 답변 누락률 | < 5% |
| 자동 답변 초안 채택률 | 60%+ |

## 3. Input / Output

**Input (전 채널)**
- Instagram (DM·댓글) — Meta Graph API
- YouTube 댓글 — Data API
- 블로그 댓글 — WordPress webhook
- X DM·멘션 — X API
- 이메일 문의 — Gmail API

**Output**
- Slack `#inbox` 채널에 통합 메시지
- AI 답변 초안 첨부
- Reaction으로 발송 (✅) / 무시 (❌) / 사람 확인 (🙋)

## 4. n8n 워크플로

```
[병렬 webhook + polling]
  ├─▶ Meta Graph (IG comments + DMs)
  ├─▶ YouTube comments polling (5분 주기)
  ├─▶ WordPress webhook (블로그 댓글)
  ├─▶ X API (멘션·DM)
  └─▶ Gmail watch (문의 라벨)
        │
        ▼
[Normalize] 공통 스키마로 변환
  {channel, author, content, url, received_at}
        │
        ▼
[Claude API: 분류 + 답변 초안]
  - intent: question / complaint / praise / spam / sales_lead
  - urgency: high / medium / low
  - sentiment: positive / neutral / negative
  - draft_reply (intent별 톤 조정)
        │
        ▼
[Notion: Inbox DB 적재]
        │
        ▼
[Slack 메시지]
  채널·작성자·내용·URL + 답변 초안
  + 액션 버튼: ✅ 발송 / 📝 수정 / ❌ 무시 / 🙋 사람
        │
        ▼
[Slack interaction]
  ✅ → 각 채널 API로 답변 발송 + Notion status=replied
  ❌ → Notion status=ignored
  🙋 → 운영팀 멘션
```

## 5. 필요한 통합

- Meta Graph API (IG Business)
- YouTube Data API
- WordPress REST API (webhook 설정)
- X API
- Gmail API
- Claude API
- Notion API
- Slack (block actions)

## 6. MVP 범위 (Week 2)

**포함**
- IG + 이메일만 (다른 채널 v2)
- 분류 + 답변 초안 + Slack 알림
- 발송은 수동 (Slack 메시지 카피)

**제외 (v2)**
- 다채널 통합
- Slack 1-click 발송 (block actions 설정)
- 자동 발송 (high confidence 답변만)

## 7. 의존성

- **독립 실행 가능**
- **시너지**: 04(경쟁사 인텔)와 동일한 Slack 운영 채널 사용

## 8. 리스크

- 잘못된 자동 답변이 브랜드 이미지 해침 → 자동 발송은 confidence 0.9+ 한정 (v2)
- API rate limit (특히 IG) → 호출 간격 조정
- 스팸 댓글 노이즈 → 1차 필터로 LLM 호출 줄임

## 9. 프롬프트

`prompts/`:
- `inbox-classify-reply.md` (분류 + 초안 한 번에)
- `tone-guide-by-channel.md`
