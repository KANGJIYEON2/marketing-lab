# linkedin-from-video

영상 분석 결과 → LinkedIn 포스트 1개.

## 모델

- 권장: `claude-sonnet-4-6`
- temperature: 0.6
- max tokens: 1200

## 시스템 메시지

```
You are writing a LinkedIn post from a YouTube video.

Brand tone: {BRAND_TONE} (slightly more professional)

Requirements:
- 1000-1500 chars
- Hook in first 2 lines (LinkedIn truncates)
- 3-4 short paragraphs separated by blank lines
- 1 personal angle / takeaway
- End with: video link + question to prompt comments
- No hashtag spam (max 3)
- Match transcript language

Return STRICT JSON:
{
  "post": "<full LinkedIn post text>"
}
```

## 사용자 메시지 (입력)

```
Video URL: {url}
Core: {core_message}

Chapters:
{JSON chapters}

Audience: {audience}
```

## LinkedIn 포스트 구조

```
[Line 1-2: 후킹 — 절단되기 전 보이는 부분]

[Paragraph 1: 맥락·계기 — "왜 이걸 했는가"]

[Paragraph 2: 핵심 인사이트 (chapters에서 발췌)]

[Paragraph 3: 개인적 시각 / 시행착오 — "내가 처음엔 X 했는데 결국 Y"]

[Line: video URL]
[Line: "여러분은 어떻게 하시나요? 댓글로 의견 주세요." 류 질문]

[Optional: #해시태그1 #해시태그2 #해시태그3]
```

## 톤 조정

LinkedIn은 X와 달리:
- 1인칭이지만 약간 더 정돈된 한국어/영어
- 농담·반어법 자제
- 회사·역할 맥락 자연스럽게 노출 (자랑 X)
- 학습·실패·인사이트 공유에 가산점

## 평가 기준

- [ ] 첫 2줄로 "더 보기" 클릭을 유도하는가
- [ ] 회사 자랑 톤이 아닌 학습 공유 톤인가
- [ ] 마지막 질문이 댓글 유도 명확한가 (yes/no 질문은 NG)
- [ ] 해시태그 3개 이하

## 흔한 실수 (LLM이 자주 빠지는 함정)

1. **이모지 남발** — 1~2개 OK, 4개 이상은 광고처럼 보임
2. **불릿 리스트** — LinkedIn에서 가독성 떨어짐. 단락 문장으로
3. **CTA 강제** — "✅ 영상 보러 가기" 같은 매크로 톤 금지. 자연스러운 한 줄
4. **너무 긴 첫 단락** — 후킹은 짧고 호기심

## 비용

- 입력: ~1.5k tokens
- 출력: ~0.8k tokens
- 포스트 1개당 ≈ $0.015
