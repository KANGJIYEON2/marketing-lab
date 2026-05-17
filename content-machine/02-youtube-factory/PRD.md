# 02. 유튜브 → 멀티채널 콘텐츠 팩토리

> 유튜브 영상 1편 업로드 → 블로그·쇼츠·X·LinkedIn·뉴스레터 자동 생성

## 1. 페인포인트

- 영상은 만들었는데 다른 채널 옮기기 귀찮아서 안 함
- 한 채널에 묶인 콘텐츠는 도달이 1배. 멀티채널은 5~7배.
- 수동 리퍼포징은 영상 1편당 3~4시간 추가 작업

## 2. KPI

| 지표 | 목표 |
|---|---|
| 영상 1편당 파생 콘텐츠 | 5~7개 |
| 수동 작업 시간 | < 30분 (편집·검수만) |
| 채널별 발행 누락률 | 0% |

## 3. Input / Output

**Input**
- 유튜브 채널 RSS 또는 Data API 신규 업로드 감지
- 영상 자막(있으면) 또는 Whisper transcribe

**Output (모두 Notion `ContentDrafts` DB에 적재)**
- 블로그 글 (1500~2500자, SEO 메타 포함)
- 쇼츠 스크립트 3개 (각 60초)
- X 스레드 (8~10개 트윗)
- LinkedIn 포스트 (1개, 1200자)
- 뉴스레터 섹션 (300~500자)

## 4. n8n 워크플로

```
[Trigger: YouTube RSS 신규 영상]
        │
        ▼
[Get Transcript]
  ├─ 자막 있으면 → YouTube API
  └─ 없으면 → 영상 다운로드 → Whisper API
        │
        ▼
[Claude API: 영상 분석]
  - 핵심 메시지 3줄
  - 챕터별 요약
  - 인용 가능한 문장 추출
        │
        ▼
[병렬 생성]
  ├─▶ Claude: 블로그 글 (SEO 최적화)
  ├─▶ Claude: 쇼츠 스크립트 3개
  ├─▶ Claude: X 스레드
  ├─▶ Claude: LinkedIn 포스트
  └─▶ Claude: 뉴스레터 섹션
        │
        ▼
[Notion: 각 채널별 행 생성, status=draft]
        │
        ▼
[Slack 알림] "영상 X의 파생 콘텐츠 5종 준비 완료 → [Notion 링크]"
        │
        ▼
[사람 검수 → status=approved]
        │
        ▼
[자동 발행]
  ├─▶ WordPress/Ghost (블로그)
  ├─▶ X API (스레드 예약)
  ├─▶ LinkedIn API
  └─▶ Mailchimp/Resend (뉴스레터 큐)
```

## 5. 필요한 통합

- YouTube Data API
- OpenAI Whisper API (자막 없는 영상용)
- Claude API
- Notion API
- WordPress REST API 또는 Ghost API
- X API v2 (포스팅)
- LinkedIn API (v2, 회사 페이지)
- Resend 또는 Mailchimp

## 6. MVP 범위 (Week 2)

**포함**
- 자막 있는 영상만 (Whisper 통합은 v2)
- 블로그 + X 스레드 + LinkedIn 만 (쇼츠/뉴스레터는 v2)
- 발행은 수동 (Notion에서 status 토글)

**제외 (v2)**
- 쇼츠 스크립트 자동 생성
- 자동 발행 (Slack 승인 → 1-click)
- 썸네일 자동 생성

## 7. 의존성

- **선행**: Layer 0, 유튜브 채널 운영 중
- **활용**: 01의 글감으로 영상 기획 → 영상 → 02 파이프라인

## 8. 리스크

- Whisper 비용 (긴 영상): 시작은 자막 있는 영상으로 한정
- LLM 응답 일관성: 채널별 톤매뉴얼 프롬프트로 고정
- API rate limit: 발행 시 큐잉

## 9. 프롬프트

`prompts/`에 채널별 분리:
- `blog-from-video.md`
- `x-thread-from-video.md`
- `linkedin-from-video.md`
- `newsletter-from-video.md`
