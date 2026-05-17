# Layer 0 — 인프라

모든 자동화가 시작 전에 한 번만 구축하는 공통 인프라.

## 파일

- [`SETUP.md`](./SETUP.md) — 9단계 셋업 가이드 (n8n·Notion·Slack·credentials)
- [`workflows/hello-world.json`](./workflows/hello-world.json) — 검증용 최소 워크플로

## 관련 문서

- [`../notion-schemas/`](../notion-schemas/) — Layer 0에서 만드는 DB 4종 스키마
- [`../../.env.example`](../../.env.example) — 환경변수 템플릿

## 완료 신호

✅ Notion DB 4개 생성
✅ Slack 채널 5개 + 봇 토큰
✅ n8n credentials 3개 등록 (Notion / Slack / Anthropic)
✅ `hello-world.json` 워크플로가 `#automation-log`에 메시지 발송 성공
✅ `.env` 파일 채워짐 + `.gitignore`에 포함됨

5개 모두 ✅면 [`/content-machine/01-idea-engine/SETUP.md`](../../content-machine/01-idea-engine/SETUP.md)의 6번부터 이어서 진행.
