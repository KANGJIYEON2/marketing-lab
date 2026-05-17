# prompts

이 자동화가 사용하는 LLM 시스템 프롬프트. 한 파일 = 한 프롬프트.

## 규칙

- 파일명: kebab-case (예: `extract-painpoint.md`, `welcome-email.md`)
- 본문 구조: 모델·temperature·시스템 메시지 본문·입출력 예시·튜닝 체크리스트
- 인라인 하드코딩 금지 — n8n 워크플로에서는 이 파일을 참조하거나 Set 노드와 동기화
