# blog-from-video

영상 분석 결과 + transcript → SEO 최적화된 블로그 초안.

## 모델

- 권장: `claude-sonnet-4-6` (긴 콘텐츠라 Opus도 검토 가능)
- temperature: 0.5 (창작성과 일관성 균형)
- max tokens: 3000

## 시스템 메시지

```
You are writing a SEO-optimized blog post derived from a YouTube video.

Brand tone: {BRAND_TONE}

Requirements:
- 1500-2500 chars body (Korean) or 1500-2500 words (English) — match transcript language
- Structure: H1 + intro (3 lines) + 3-5 H2 sections + conclusion
- Use the chapters and pull_quotes from input as anchors
- Embed the original video link near top: <iframe-style markdown link>
- End with one CTA line

Return STRICT JSON:
{
  "meta_title": "<<=60 chars>",
  "meta_desc": "<<=155 chars>",
  "body_markdown": "<full body, no front-matter>"
}
```

`{BRAND_TONE}`은 `Config.brand_tone`.

## 사용자 메시지 (입력)

```
Video URL: {url}
Title: {original title}
Core: {core_message}
Target keyword: {target_keyword}
Audience: {audience}

Chapters:
{JSON chapters}

Pull quotes:
- {quote 1}
- {quote 2}
- {quote 3}

Transcript (truncate ok):
{transcript first 8000 chars}
```

Transcript는 워크플로에서 8000자로 자름 — 블로그 글 자체에 필요한 디테일은 분석 단계에서 이미 추출됨.

## 출력 가이드

### meta_title 규칙
- 60자 이내
- target_keyword 자연스럽게 포함
- 클릭률 유도하는 후킹 (숫자·how-to·핵심 약속)

### meta_desc 규칙
- 155자 이내
- 문제 → 해결 약속 구조
- target_keyword 1회 포함

### body_markdown 규칙
- 첫 단락에 video link: `[원본 영상 보기](url)`
- H2 섹션 제목은 chapters의 title 활용 가능 (단, blog에 맞게 다듬을 것)
- pull_quotes는 `> 인용` 블록으로 활용
- 마지막 CTA 한 줄 — 채널 구독 / 다음 글 / 뉴스레터 등 1개만

## 평가 기준

- [ ] meta_title이 클릭하고 싶은가 (단순 영상 제목 복사 X)
- [ ] body가 영상 안 보고도 가치 전달되는가 (영상 보조 자료가 아닌 독립 콘텐츠)
- [ ] target_keyword가 본문에 3~5회 자연스럽게 등장하는가
- [ ] CTA가 명확한가

체크리스트 70% 미달 → 시스템 프롬프트 보강 (특히 brand_tone).

## 비용

- 입력: ~9k tokens (transcript + analysis + system)
- 출력: ~2.5k tokens
- 블로그 1편당 ≈ $0.07
