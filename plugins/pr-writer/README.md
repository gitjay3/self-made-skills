# PR Writer

브랜치의 git 컨텍스트를 분석해 구조화된 GitHub PR 제목·본문을 자동 작성하는 Claude Code 플러그인입니다.

## 기능

- 브랜치 전체 커밋·diff를 종합 분석 (마지막 커밋이 아닌 누적 결과)
- 능동태 현재형 동사 PR 제목 (영어 기본, 한국어 옵션)
- 한국어 PR 본문 (Why / Approach / How it works / Links)
- **`.github/pull_request_template.md` 자동 감지·차용** (다중 템플릿도 지원)
- **`Closes #` / `Fixes #` / `Resolves #` 키워드로 이슈 자동 닫기**
- **`--draft` / `--reviewer` / `--label` / `--assignee` 플래그 지원**
- **본인 자동 assign** (`assignee:none`으로 비활성화)
- **Co-author 자동 검출** (다중 작성자 PR에 인간 contributor 푸터)
- **민감 정보 자동 스캔** (50+ 서비스 토큰 패턴, 위험 파일 차단, PII 검출, PR 제목·HTML 주석까지 전 영역 검사 → 발견 시 작성 중단·즉시 rotate 권고)
- 신규 PR 생성 + 기존 PR 본문 갱신 모드 자동 분기
- 푸시 안 된 커밋 자동 감지·안내
- AI/Claude 표현·`Co-Authored-By: Claude` 자동 배제

<br>

## 설치

### 마켓플레이스에서 설치

```bash
/plugin marketplace add gitjay3/self-made-skills
/plugin install pr-writer@self-made-skills
```

요구사항: GitHub CLI (`gh`)가 설치·인증되어 있어야 합니다 (`gh auth login`).

<br>

## 사용법

### 기본 호출

```bash
# 기본 (base: main, 본문 한국어, 본인 자동 assign)
/pr-writer:write

# 다른 base 브랜치
/pr-writer:write develop

# 본문도 영어로
/pr-writer:write main language: en
```

### 고급 옵션 조합

```bash
# Draft PR + 리뷰어 지정
/pr-writer:write main draft reviewer:alice,bob/team-frontend

# 라벨 + 이슈 자동 닫기
/pr-writer:write main label:bug,backend closes:#142

# 자동 assign 비활성화
/pr-writer:write main assignee:none

# 다른 사람에게 assign
/pr-writer:write main assignee:alice
```

### 인자 표

| 옵션 | 패턴 | 기본값 |
|------|------|--------|
| `base-branch` | 첫 번째 토큰 (`main`, `develop` 등) | `main` |
| `language` | `language: ko` 또는 `language: en` | `ko` |
| `draft` | `draft` (단독) | false |
| `reviewer` | `reviewer:user1,user2,org/team` | 없음 |
| `label` | `label:bug,backend,docs` | 없음 |
| `closes` | `closes:#123` 또는 `closes:#123,#456` | 없음 |
| `assignee` | `assignee:user1` 또는 `assignee:none` | `@me` (본인) |

### 자연어 호출

다음과 같은 요청에도 반응:

```
PR 올려줘
PR 본문 써줘
풀리퀘 만들어줘
```

기존 PR이 이미 열려있다면 **업데이트 모드**로 전환되어 본문을 현재 상태에 맞게 다시 작성합니다.

<br>

## 동작 흐름

1. **컨텍스트 자동 수집** — git status / branch / log / contributors / gh user / open PR / **PR 템플릿 파일**
2. **Pre-flight 체크** — gh 인증, 푸시 안 된 커밋, 기존 PR 존재 여부
3. **diff 분석** — base 브랜치 대비 누적 변경
4. **민감 정보 스캔** — API 키·토큰·비밀번호·프라이빗 키·PII 발견 시 작성 중단·경고
5. **제목 생성** — 능동태 현재형 동사
6. **본문 작성** — 레포 PR 템플릿 우선, 없으면 기본 구조
7. **검토 + 실행** — `gh pr create` 또는 `gh pr edit`

<br>

## 출력 예시

### PR 제목

```
Add user authentication with JWT
```

### PR 본문 (이슈 닫기 + Co-author 포함)

````markdown
Closes #142

## Why

기존 세션 기반 인증은 모바일 클라이언트와 외부 API 통합에서 한계가 있었음.

## Approach

JWT(`jsonwebtoken`)로 stateless 인증을 도입하고, refresh token을 별도 저장소(Redis)에 둔다.
- 검토안: 세션 + 쿠키 → 모바일 호환성 부족으로 기각
- 검토안: OAuth 위임 → 자체 사용자 DB가 있어 과도

## How it works

- `auth` 모듈 신규 추가: `signToken`, `verifyToken`, `refreshToken`
- 미들웨어 `requireAuth`로 보호 라우트 적용
- Redis에 refresh token 저장 (TTL 14일)
- 의존성 추가: `jsonwebtoken`, `ioredis`

## Links

- 참고: https://datatracker.ietf.org/doc/html/rfc7519

---

Co-authored-by: Alice <alice@example.com>
````

<br>

## PR 템플릿 자동 감지

레포에 다음 파일이 있으면 그 구조를 그대로 따라 작성합니다:

- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `docs/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE/` (다중 템플릿 — 사용자가 선택)

템플릿이 없으면 기본 구조 (Why / Approach / How it works / Links) 사용.

<br>

## 핵심 룰 (위반 금지)

### 보안·프라이버시 (최우선)

- 🛑 **Security Filter 절대 스킵 금지** — diff/커밋 메시지/PR 제목/브랜치 이름/HTML 숨김 영역까지 전부 검사
- 🛑 **위험 파일 차단** — `.env`, `*.pem`, `*.key`, `id_rsa`, `credentials.json`, `*.sql.dump` 등이 diff에 있으면 PR 작성 중단
- **민감 정보 본문 노출 금지** — API 키, 토큰, 비밀번호, 프라이빗 키, DB 연결 문자열, Webhook URL (위치만 표기 또는 `<REDACTED>` 마스킹)
- **부분 인용·해시 인용도 금지** — `sk-abc...xyz` (앞뒤 일부도 추론 가능), SHA256 해시도 brute-force 가능
- **내부 인프라 정보 노출 금지** — 사설 IP, staging endpoint, 내부 도메인(`*.internal.`/`*.corp.`/`*.local`), localhost:포트
- **PII(개인정보) 노출 금지** — 사용자 이메일·신용카드·주민번호·외국인등록번호·전화번호·운전면허·여권번호 (git commit author 이메일은 예외)
- **PR 제목·커밋 메시지에 secret 금지**
- **HTML 주석·Markdown 숨김 영역에 secret 금지**
- ⚠️ **public repo에서 secret 발견 시 즉시 rotate** — 봇이 수 분 내 수집함

### 스타일·내용

- AI/Claude 관련 표현 절대 금지 ("Generated with Claude" 등)
- `Co-Authored-By: Claude` 헤더 금지 (인간 contributor만 허용)
- 이모지·아이콘 금지 (명시 요청 시만)
- "이번 PR에서는 X를 추가했습니다" 같은 변경 로그식 표현 금지 — 현재 상태 진술로
- 빈 섹션 금지 — 해당 없으면 섹션 통째로 생략

<br>

## Security Filter (다층 가드)

### 4단계 검사

1. **위험 파일 차단** — `.env`, `*.pem`, `*.key`, `id_rsa`, `credentials.json`, `*.kubeconfig`, `*.sql.dump`, `*.csv`(사용자 데이터) 등이 diff에 들어 있으면 즉시 stop
2. **Secret 패턴 50+ 검출** — 아래 표 참조
3. **PII 검출** — 한국·미국 PII 패턴 (주민번호, 외국인등록번호, 신용카드, SSN 등)
4. **숨김 영역 검사** — HTML 주석, Markdown 링크 hidden URL, 제로폭 문자, PR 제목·브랜치 이름

### 검출 패턴 50+

| 카테고리 | 서비스 |
|---|---|
| **클라우드** | AWS, GCP service account, Azure connection string, Firebase, Cloudflare, DigitalOcean, Heroku |
| **AI/ML** | OpenAI, Anthropic, Hugging Face, Replicate, Google AI |
| **VCS/Package** | GitHub PAT/OAuth/App, GitLab, npm, PyPI, Docker Hub PAT |
| **메시징** | Slack(bot/webhook), Discord(bot/webhook), Telegram bot |
| **결제·메일** | Stripe, Twilio, SendGrid, Mailgun, Mailchimp |
| **생산성** | Notion, Linear |
| **인증 토큰** | JWT, Bearer, Basic Auth, OAuth refresh/access token |
| **DB 연결** | PostgreSQL, MongoDB(SRV 포함), MySQL, Redis, AMQP, MSSQL, JDBC |
| **프라이빗 키** | RSA/EC/DSA/OpenSSH PRIVATE KEY 블록 |
| **사설 네트워크** | 내부 도메인(`*.internal.`/`*.corp.`/`*.local`), 사설 IP, staging URL |
| **PII (한국)** | 주민번호, 외국인등록번호, 전화번호, 운전면허, 여권번호 |
| **PII (글로벌)** | 신용카드(Luhn), 미국 SSN, 사용자 이메일 dump |

### False Positive 처리

자동 통과:
- 예시 파일 (`*.example.*`, `*.sample.*`, `*.template`)
- placeholder (`your-api-key-here`, `xxx`, `change-me`, `INSERT_KEY_HERE`)
- 테스트 디렉토리의 명백한 가짜 값 (`test_key_1234`, `dummy`, `fake-`, `mock-`)
- 주석 직후 패턴 (`# example`, `// fake`)
- 이미 `.gitignore`된 파일

모호하면 사용자에게 확인 후 처리.

### 발견 시 권고

```
긴급 권고 (public repo면 더더욱):
1. 노출된 키는 지금 즉시 rotate — 봇이 수 분 내 수집함
2. 모든 사용 서비스 monitoring 강화
3. secret을 환경변수/secret manager로 이동 (.env는 .gitignore에)
4. git history에서 제거: git filter-repo --invert-paths --path <file>
5. force push 전 팀에 공지 (collaborators 영향)
6. 회사 보안팀에 사고 보고 (PII 노출 시 필수)
```

### ⚠️ 한계 인지 — 도구와 함께 사용

LLM 기반 단일 검사는 false negative 가능. **반드시 자동화 도구와 함께**:

| 도구 | 위치 | 장점 |
|---|---|---|
| [gitleaks](https://github.com/gitleaks/gitleaks) | pre-commit hook | 빠름, 150+ 패턴 |
| [TruffleHog](https://github.com/trufflesecurity/trufflehog) | CI/CD | 700+ detector + 실제 API 검증 |
| [GitHub Secret Scanning](https://docs.github.com/code-security/secret-scanning) | repo Settings | Push protection 활성화 권장 |
| [GitGuardian](https://www.gitguardian.com/) | 실시간 monitoring | 다국어 지원 |

이 스킬은 **마지막 안전망**. 1차 방어는 위 도구들이 담당.

<br>

## 언어 정책

- **제목**: 영어 우선. `language: ko` 인자로 한국어.
- **본문**: 한국어 기본. `language: en` 인자로 영어.
- 기술 용어는 영어 원문 그대로 (`lifetime`, `lazy loading`, `race condition` 등)

<br>

## 라이선스

MIT
