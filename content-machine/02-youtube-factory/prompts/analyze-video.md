# analyze-video

영상 transcript를 받아 다른 채널 콘텐츠 생성에 필요한 **공통 컨텍스트**(빌딩 블록)를 추출하는 전처리 프롬프트.

이 프롬프트의 출력은 blog/x-thread/linkedin 3개 프롬프트에 모두 input으로 들어간다. 따라서 LLM 호출은 영상당 4번이 아니라 1+3번.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.3 (분석은 일관성 우선)
- max tokens: 1500

## 시스템 메시지

```
You are analyzing a YouTube video transcript to extract reusable content building blocks.

Return STRICT JSON:
{
  "core_message": "<one-sentence thesis of the video>",
  "chapters": [{"title":"...","summary":"..."}],
  "pull_quotes": ["...","...","..."],
  "target_keyword": "<one from: {KEYWORD_POOL}, or 'other'>",
  "audience": "<persona in one phrase>"
}

No prose, no markdown, no code fences. Output language: match transcript language.
```

`{KEYWORD_POOL}`은 워크플로 `Config` 노드의 `target_keyword_pool` 배열이 join으로 삽입.

## 사용자 메시지 (입력)

```
Title: {video title}
Description: {video description}

Transcript:
{full transcript}
```

긴 영상의 경우 transcript는 워크플로에서 자르지 않음 — Claude의 200k 컨텍스트로 충분 (1시간 영상 ≈ 9k 토큰).

## 출력 예시

```json
{
  "core_message": "1인 창업자가 n8n으로 마케팅 자동화 10종을 12주에 적층 구축한 실전 과정.",
  "chapters": [
    {"title": "왜 자동화인가", "summary": "수동 운영의 한계와 ROI 비교"},
    {"title": "Layer 0 인프라", "summary": "Notion + n8n + Slack 셋업 시간 1일"},
    {"title": "콘텐츠 머신부터 시작한 이유", "summary": "다른 레이어의 input이 콘텐츠라서"},
    {"title": "실제 비용 공개", "summary": "월 $50 미만으로 운영"}
  ],
  "pull_quotes": [
    "자동화를 한 번에 5개 시작하면 0개가 완성된다",
    "Notion은 워크플로가 아니라 상태 저장소다",
    "휴먼-인-더-루프를 디폴트로 둬야 신뢰가 쌓인다"
  ],
  "target_keyword": "n8n automation",
  "audience": "1인 SaaS 창업자 / 인하우스 마케터"
}
```

## 평가 기준

- [ ] `core_message`가 영상을 모르는 사람도 이해할 정도로 self-contained
- [ ] `chapters`는 3~6개. 2개 이하면 영상이 짧거나 LLM이 게으름. 8개 이상이면 너무 잘게 쪼갬
- [ ] `pull_quotes`는 그대로 인용해도 매력적인 문장
- [ ] `target_keyword`는 키워드 풀에 실제 있거나 `other`

## 비용

- 입력: transcript에 따라 다름. 평균 5k tokens
- 출력: ~800 tokens
- 영상 1편당 ≈ $0.03 (Sonnet 4.6 기준)
