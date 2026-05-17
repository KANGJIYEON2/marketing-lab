# marketing-lab

In-house 마케팅 자동화 10종을 단계적으로 구축하는 모노레포. n8n 워크플로 엔진을 중심으로, AI(Claude/OpenAI)를 결합해 콘텐츠·매출·운영 전 영역을 자동화한다.

---

## 1. 목적

1인 또는 소규모 팀이 콘텐츠/SEO + SNS + 유료광고 + 이메일/CRM을 **사람 손 없이 굴러가는 시스템**으로 만든다. 각 자동화는 독립 실행 가능하지만, 단일 콘텐츠/리드/성과 DB를 공유해 서로의 input/output이 된다 (compound effect).

---

## 2. 폴더 구조

모노레포. 각 자동화는 동일한 서브폴더 구조를 가진다.

```
marketing-lab/
├── CLAUDE.md                 # 이 파일 (전체 컨텍스트)
├── README.md                 # 자동화 인덱스
├── .env.example              # 환경변수 템플릿 (전 자동화 공유)
├── .gitignore
├── docs/                     # 레포 전역 문서
│   ├── README.md
│   ├── roadmap.md            # 12주 실행 로드맵
│   ├── layer-0/              # Week 1 인프라 (모든 자동화의 전제)
│   │   ├── README.md
│   │   ├── SETUP.md          # 9단계 셋업 가이드
│   │   └── workflows/
│   │       └── hello-world.json
│   └── notion-schemas/       # DB 4종 스키마 (자동화들이 공유)
│       ├── README.md
│       ├── content-ideas.md
│       ├── content-drafts.md
│       ├── leads.md
│       └── campaign-metrics.md
├── content-machine/          # 콘텐츠 생산·확산
│   ├── README.md
│   ├── 01-idea-engine/
│   ├── 02-youtube-factory/
│   ├── 05-seo-hunter/
│   └── 08-ugc-collector/
├── revenue-impact/           # 매출 직결
│   ├── README.md
│   ├── 06-ad-evolution/
│   ├── 07-lead-sequence/
│   └── 10-lp-ab-test/
└── ops-reduction/            # 운영 부담 감소
    ├── README.md
    ├── 03-unified-inbox/
    ├── 04-competitor-intel/
    └── 09-weekly-report/
```

### 자동화 폴더 표준 구조

각 `XX-name/` 자동화 폴더는 다음을 가진다:

```
XX-name/
├── README.md       # 자동화 진입점 (한 줄 설명·상태·진입점 링크)
├── PRD.md          # 제품 요구사항
├── SETUP.md        # 셋업 가이드 (구현 단계에서 작성)
├── prompts/        # LLM 시스템 프롬프트 (.md)
├── workflows/      # n8n export JSON
├── scripts/        # 보조 스크립트 (필요 시)
└── docs/           # 자동화 전용 추가 문서
```

서브폴더 자체에도 README.md가 있어 역할과 파일 작명 규칙을 안내한다.

---

## 3. 기술 스택

| 영역 | 도구 | 비고 |
|---|---|---|
| 워크플로 엔진 | **n8n** (셀프호스팅) | 모든 자동화의 중앙 허브 |
| LLM | Claude API (Sonnet 4.6 default, Opus 4.7 복잡 작업) | 보조: OpenAI gpt-5-mini |
| 콘텐츠 DB | **Notion** | 글감·초안·발행·성과 단일 SoT |
| 리드/CRM DB | Notion 또는 Airtable | 검토 후 결정 |
| 자산 저장소 | Google Drive | 이미지·영상·캐러셀 |
| 분석 | GA4, Meta Marketing API, Google Ads API, GSC | n8n에서 직접 호출 |
| 알림 | Slack | 모든 자동화의 휴먼-인-더-루프 채널 |
| 이메일 발송 | Resend 또는 Mailchimp (검토) | |
| 발행 | WordPress/Ghost (블로그), Meta Graph API (IG/Threads), X API, LinkedIn API | |

---

## 4. 공통 의존성

이 자동화들이 공유하는 인프라. **Layer 0 (1주차)에 한 번만 구축**한다.

- **Notion DB 4종**: ContentIdeas, ContentDrafts, Leads, CampaignMetrics
- **n8n credentials**: Claude/OpenAI 키, Notion 토큰, Slack webhook, Meta/Google Ads 토큰, GA4 서비스 계정
- **공통 webhook URL 컨벤션**: `/{automation-id}/{event}` (예: `/02-youtube-factory/new-video`)
- **환경변수 관리**: `.env` (gitignore), 프로덕션은 n8n credentials 매니저

---

## 5. 개발 원칙

1. **MVP 먼저, 그 다음 적층**: 각 자동화는 1주 안에 끝나는 MVP 범위로 시작. 욕심내면 발행이 안 됨.
2. **휴먼-인-더-루프 디폴트**: 첫 버전은 항상 Slack 승인 단계 포함. 신뢰 쌓이면 자동화 비율 ↑.
3. **데이터는 Notion에 영속**: n8n은 워크플로만, 상태는 Notion에. 워크플로 재실행 가능해야 함.
4. **프롬프트 버전 관리**: 각 자동화 폴더 `prompts/` 디렉토리에 마크다운으로 보관. 인라인 하드코딩 금지.
5. **관찰 가능성**: 모든 실행은 Slack `#automation-log` 채널에 결과 요약 송출.

---

## 6. 자동화 10종 요약

### 콘텐츠 머신 (content-machine)
- **01. 아이디어 엔진** — Reddit/X/유튜브 댓글에서 페인포인트 수집 → 글감 DB
- **02. 유튜브 팩토리** — 영상 1편 → 블로그+쇼츠+X+LinkedIn+뉴스레터 자동 변환
- **05. SEO 헌터** — GSC 11~20위 키워드 자동 발굴 → 업데이트 액션 생성
- **08. UGC 컬렉터** — 리뷰·태그·멘션 수집 → 캐러셀·광고·LP 소재화

### 매출 영향 (revenue-impact)
- **06. 광고 진화 엔진** — 주간 광고 성과 분석 → 신규 변형 자동 생성·업로드
- **07. 리드 시퀀스** — 폼 제출 → enrich → 개인화 환영 메일 3단
- **10. LP A/B 테스트** — 헤드라인·CTA 변형 자동 생성·배포·위너 채택

### 운영 부담 (ops-reduction)
- **03. 통합 인박스** — IG·YT·블로그·메일 댓글 통합 → AI 분류·답변 초안
- **04. 경쟁사 인텔** — 경쟁사 블로그·SNS·광고·가격 변화 일간 모니터링
- **09. 주간 리포트** — GA4+광고+이메일+SNS 통합 → 액션 추천 → 월요일 Slack

상세 명세는 각 폴더의 `PRD.md` 참고.

---

## 7. 작업 순서 권장

[`docs/roadmap.md`](./docs/roadmap.md)에 12주 단계별 일정 정리. 핵심 순서:

1. **Layer 0 (1주)**: 공통 인프라 — [`docs/layer-0/SETUP.md`](./docs/layer-0/SETUP.md) 따라 진행
2. **콘텐츠 머신** 먼저: 01 → 02 → 05 → 08 (글감과 콘텐츠 자산이 다른 레이어의 input)
3. **매출 영향**: 06 → 07 → 10
4. **운영 부담**: 03 → 04 → 09 (병렬 진행 가능)

의존성 그래프:
- 02·05·08 → 01(아이디어/주제) 출력 활용 가능
- 06·10 → 02(콘텐츠 자산), 09(성과 데이터) 활용
- 09 → 모든 자동화의 성과 로그 집계

---

## 8. 작업할 때 Claude에게

- **이 파일을 먼저 읽고** 어떤 자동화 범위인지 파악할 것
- 새 자동화 추가 시 **자동화 폴더 표준 구조**(섹션 2) 그대로 따를 것
- 각 자동화 폴더에 `README.md`(진입점) + `PRD.md`(요구사항) 최소 2개
- **n8n 워크플로는 JSON export로** `workflows/` 폴더에 저장
- 모든 LLM 프롬프트는 `prompts/*.md`로 분리 (인라인 금지)
- **Notion DB 스키마 변경 시** [`docs/notion-schemas/`](./docs/notion-schemas/)와 영향받는 자동화 PRD 모두 업데이트
- **개발 원칙 5가지**(섹션 5) 항상 준수
