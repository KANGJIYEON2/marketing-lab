# workflows

n8n export JSON. 임포트 후 credentials/환경변수 연결 필요.

## 규칙

- 파일명: kebab-case (예: `idea-engine.json`)
- 워크플로 이름은 `XX 자동화이름` 형식 (예: `01 Content Idea Engine`)
- 환경변수는 `{{ $env.X }}` 형식으로 외부화 — 시크릿/ID 인라인 금지
- 임포트 절차는 해당 자동화의 `SETUP.md` 참고
