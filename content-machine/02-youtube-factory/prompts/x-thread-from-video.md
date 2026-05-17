# x-thread-from-video

영상 분석 결과 → X(Twitter) 스레드 8~10개 트윗.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.6 (X는 후킹·재치가 중요해 약간 높게)
- max tokens: 1500

## 시스템 메시지

```
You are writing an X (Twitter) thread from a YouTube video.

Brand tone: {BRAND_TONE}

Requirements:
- 8-10 tweets total
- Tweet 1: strong hook (<280 chars), no link
- Tweets 2-N: one insight per tweet, each <=280 chars
- Last tweet: include the video link + 1-line CTA
- Use chapters as content backbone
- Match transcript language

Return STRICT JSON:
{
  "tweets": ["tweet 1", "tweet 2", ...]
}
```

## 사용자 메시지 (입력)

```
Video URL: {url}
Core: {core_message}

Chapters:
{JSON chapters}

Pull quotes:
- {quote 1}
- ...
```

Transcript는 X 스레드 생성엔 불필요 — 분석 단계의 chapters + pull_quotes로 충분.

## 후킹 (Tweet 1) 패턴 권장

LLM 출력의 첫 트윗이 이 중 하나의 패턴을 따르면 좋음 (프롬프트에 명시하진 않음 — 다양성 확보):

1. **반직관 주장**: "마케팅 자동화 10개 만들었더니, 시간이 더 들었습니다. 이유는—"
2. **숫자 후킹**: "12주 만에 자동화 10개. 비용 월 $47. 회수 시간 22시간/주. 어떻게 했나"
3. **실수 공개**: "처음엔 모두 동시에 시작했어요. 6주 뒤 0개 완성. 다시 짠 순서가—"
4. **고민 → 해결**: "1인 마케터 가장 큰 적은 '오늘 뭐 쓰지?' — 이걸 죽이는 자동화 1개부터"

LLM이 매번 다양한 패턴을 시도하도록 temperature 0.6 유지.

## 트윗 본문 규칙

- **한 트윗 = 한 인사이트** (담론 늘리기 X)
- 숫자·구체적 예시 권장
- 인용 부호 사용 시 → `pull_quotes`에서 가져온 문장
- 이모지는 0~1개 정도. 과하면 X 알고리즘 노이즈

## 마지막 트윗 (CTA)

```
{N}/ 전체 흐름은 영상에서 자세히:
[video URL]

다음 주에는 '{다음 토픽}' 다룰 예정.
구독하시면 놓치지 않습니다.
```

위는 예시. LLM이 chapters와 audience에 맞춰 변형.

## 평가 기준

- [ ] Tweet 1의 후킹이 스크롤 멈추게 하는가 (테스트: 친구에게 보여줘 반응)
- [ ] 각 트윗이 독립적으로도 가치 있는가 (스레드 안 펴봐도 의미 전달)
- [ ] 마지막 트윗 외에 링크/태그 남발하지 않는가
- [ ] 영상 안 보고도 이해되는가

## 비용

- 입력: ~2k tokens
- 출력: ~1k tokens
- 스레드 1개당 ≈ $0.02
