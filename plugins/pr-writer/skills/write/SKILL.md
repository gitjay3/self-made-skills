---
name: write
description: This skill should be used when the user asks to "PR 올려줘", "PR 본문 써줘", "풀리퀘 만들어줘", "PR 만들어", "PR 갱신", "PR 본문 갱신", "create a PR", "open a pull request", "update PR description", "rewrite PR body", or wants to push a feature branch and open/update a GitHub pull request. Generates PR title (active voice) and body from full git context using gh CLI. Auto-detects existing PR (update mode), repository PR template (.github/pull_request_template.md), and human co-authors. Supports draft/reviewer/label/assignee options and "Closes #" issue keywords.
argument-hint: "[base-branch] [language: ko|en] [draft] [reviewer:user1,team] [label:bug,backend] [closes:#123]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git branch:*), Bash(git log:*), Bash(git push:*), Bash(gh auth status:*), Bash(gh pr view:*), Bash(gh pr create:*), Bash(gh pr edit:*), Bash(gh api:*), Bash(ls:*), Bash(cat:*)
---

# PR Writer

Create or update a GitHub PR with a structured title and body, generated from the current branch's git context. Auto-adapts to repository PR template, detects co-authors, and supports labels/reviewers/draft.

## Auto-Collected Context

Gathered automatically before this skill runs:

- **Status**: !`git status --short`
- **Branch**: !`git branch --show-current`
- **Recent commits**: !`git log --oneline -20`
- **Contributors**: !`git log --pretty=format:'%an <%ae>' main..HEAD 2>/dev/null | sort -u || echo "NO_RANGE"`
- **GH auth**: !`gh auth status 2>&1 || echo "NOT_AUTHENTICATED"`
- **GH user**: !`gh api user --jq .login 2>/dev/null || echo "UNKNOWN"`
- **Open PR**: !`gh pr view --json number,title,body,headRefName,baseRefName 2>/dev/null || echo "NO_OPEN_PR"`
- **PR Template (single)**: !`for f in .github/pull_request_template.md .github/PULL_REQUEST_TEMPLATE.md docs/pull_request_template.md; do [ -f "$f" ] && cat "$f" && exit; done; echo "NO_TEMPLATE"`
- **PR Templates (multi)**: !`ls .github/PULL_REQUEST_TEMPLATE/ 2>/dev/null || echo "NO_MULTI"`

> 만약 base가 `main`이 아닌 다른 브랜치라면 Contributors 명령은 잘못된 결과를 낼 수 있음. 인자 파싱 후 실제 base로 한 번 더 실행.

## Arguments

User passed: `$ARGUMENTS`

Parse the following options:

| 옵션 | 패턴 | 기본값 |
|------|------|--------|
| `base-branch` | 첫 번째 토큰 (예: `main`, `master`, `develop`) | `main` |
| `language` | `language: ko` 또는 `language: en` | `ko` |
| `draft` | `draft` 키워드 단독 | false |
| `reviewer` | `reviewer:user1,user2,org/team` | 없음 |
| `label` | `label:bug,backend,docs` | 없음 |
| `closes` | `closes:#123` 또는 `closes:#123,#456` | 없음 |
| `assignee` | `assignee:user1` 또는 `assignee:none` | `@me` (본인 자동) |

## Workflow

### Step 1: Pre-flight Check

- If GH auth is `NOT_AUTHENTICATED`, instruct user to run `gh auth login` and stop.
- 푸시되지 않은 커밋 있으면 안내: "푸시되지 않은 커밋이 있어요. `git push` 먼저 진행할까요?"
- Open PR이 `NO_OPEN_PR`이 아니면 **update mode**로 전환.

### Step 2: Diff Against Base

Run `git diff {base-branch}...HEAD` via Bash. base가 main이 아니면 Contributors도 재실행.

### Step 3: 변경 분석

전체 커밋과 diff를 종합하여 식별:
- **What/Why/How/Links** — 누적 결과 기준 (마지막 커밋만 보지 말 것)

### Step 4: 민감 정보 스캔 (Security Filter)

> **2024년 GitHub에서만 39M+ secrets 유출**, 2025년 28M+ — 매년 증가.
> Toyota(5년간 노출 → 27만 고객), GhostAction(3,325 secrets), tj-actions(23,000 repos),
> s1ngularity(PR title injection)... 한 번 push되면 git history에 영구 박힘.
> **이 단계는 절대 스킵하지 말 것.**

#### 4a. 스캔 대상 (모든 영역 검사)

s1ngularity / Copilot prompt injection 사고 교훈: 본문만이 아니라 **모든 텍스트 영역** 검사:

- **diff 전체** — 추가된 코드/파일 내용
- **커밋 메시지** (subject + body)
- **PR 제목** — s1ngularity는 PR title의 unsanitized injection이 트리거
- **브랜치 이름** — 가끔 secret 포함
- **이슈 본문/링크** (참조하는 이슈 분석 시)
- **HTML/Markdown 숨김 영역** — Copilot 사고: `<!-- -->` 주석 안에 secret 숨김

#### 4b. 위험 파일 자동 차단 (즉시 stop)

다음 파일이 diff에서 **새로 추가**되거나 **변경**됐으면 작성 중단:

| 카테고리 | 파일 패턴 |
|---|---|
| 환경 변수 실제값 | `.env`, `.env.local`, `.env.production`, `.env.staging`, `.envrc` |
| 프라이빗 키 | `*.pem`, `*.key`, `*.pfx`, `*.p12`, `*.jks`, `id_rsa`, `id_dsa`, `id_ecdsa`, `id_ed25519` |
| 인증 자격증명 | `credentials.json`, `secrets.yml`, `secrets.yaml`, `auth.json`, `*.kubeconfig`, `kubeconfig` |
| 클라우드 자격증명 | `.aws/credentials`, `.aws/config`, `gcp-key.json`, `service-account*.json`, `*-firebase-adminsdk-*.json` |
| 데이터 덤프 | `*.sql.dump`, `*.sql.gz`, `*.csv` (사용자 데이터 가능성), `*.bak`, `dump.rdb` |
| Cookie 파일 | `cookies.txt`, `*.har` (HTTP Archive — auth header 포함 가능) |

> `.env.example`, `.env.template`, `*.template`, `*.sample`은 통과 (예시 파일).

#### 4c. 검출 패턴 (Secret Patterns)

Gitleaks 150+ / TruffleHog 700+ 패턴 중 **현실에서 자주 유출되는 핵심 50+** 적용:

##### API 키 / 토큰

| 서비스 | 패턴 |
|---|---|
| **AWS Access Key** | `AKIA[A-Z0-9]{16}` |
| **AWS Secret Key** | 40자 base64 문자열 + `aws_secret` 키워드 근처 |
| **GitHub PAT** | `ghp_[A-Za-z0-9]{36}`, `gho_[A-Za-z0-9]{36}`, `ghs_[A-Za-z0-9]{36}`, `ghu_[A-Za-z0-9]{36}`, `github_pat_[A-Za-z0-9_]{82}` |
| **GitHub OAuth** | `gho_[A-Za-z0-9]{36}` |
| **OpenAI** | `sk-[A-Za-z0-9]{48}`, `sk-proj-[A-Za-z0-9_-]{80,}` |
| **Anthropic** | `sk-ant-[A-Za-z0-9_-]{95,}` |
| **Google API** | `AIza[A-Za-z0-9_-]{35}` |
| **Google OAuth** | `[0-9]+-[A-Za-z0-9_]{32}\.apps\.googleusercontent\.com` |
| **Stripe** | `sk_live_[A-Za-z0-9]{24,}`, `pk_live_[A-Za-z0-9]{24,}`, `rk_live_[A-Za-z0-9]{24,}` |
| **Slack Bot/User** | `xox[baprs]-[0-9]+-[0-9]+-[A-Za-z0-9]{24,}` |
| **Slack Webhook** | `https://hooks\.slack\.com/services/T[A-Z0-9]+/B[A-Z0-9]+/[A-Za-z0-9]+` |
| **Discord Webhook** | `https://discord(app)?\.com/api/webhooks/\d+/[A-Za-z0-9_-]+` |
| **Discord Bot** | `[MN][A-Za-z\d]{23}\.[A-Za-z\d_-]{6}\.[A-Za-z\d_-]{27,}` |
| **Telegram Bot** | `\d{8,10}:[A-Za-z0-9_-]{35}` |
| **Twilio** | `SK[a-f0-9]{32}`, `AC[a-f0-9]{32}` |
| **SendGrid** | `SG\.[A-Za-z0-9_-]{22}\.[A-Za-z0-9_-]{43}` |
| **Mailgun** | `key-[a-f0-9]{32}` |
| **Mailchimp** | `[a-f0-9]{32}-us\d{1,2}` |
| **npm Token** | `npm_[A-Za-z0-9]{36}` |
| **Notion API** | `secret_[A-Za-z0-9]{43}` |
| **Linear API** | `lin_api_[A-Za-z0-9]+` |
| **Hugging Face** | `hf_[A-Za-z0-9]{34}` |
| **Replicate** | `r8_[A-Za-z0-9]{37}` |
| **DigitalOcean** | `dop_v1_[a-f0-9]{64}` |
| **PyPI Token** | `pypi-AgEIcHlwaS5vcmc[A-Za-z0-9_-]{50,}` |
| **Docker Hub PAT** | `dckr_pat_[A-Za-z0-9_-]{27,}` |
| **Cloudflare** | `[A-Za-z0-9_-]{37}` + 컨텍스트 키워드 |

##### 인증 토큰 (일반)

- **JWT**: `eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+`
- **Bearer**: `Authorization:\s*Bearer\s+[A-Za-z0-9_.-]{20,}`
- **Basic Auth**: `Authorization:\s*Basic\s+[A-Za-z0-9+/=]{20,}`
- **OAuth refresh/access**: `(refresh_token|access_token)\s*[:=]\s*["'][^"']+["']`

##### 클라우드 자격증명 블록

- **Azure Connection String**: `DefaultEndpointsProtocol=https;AccountName=[^;]+;AccountKey=[^;]+`
- **GCP Service Account JSON**: `"type":\s*"service_account"` + `"private_key":\s*"-----BEGIN`
- **Firebase Config**: `"apiKey":\s*"AIza[A-Za-z0-9_-]+",\s*"authDomain":` (공개 정보지만 `.env`에 있으면 위험)

##### 비밀번호 / 시크릿 키 / 프라이빗 키

- **비밀번호 평문**: `(password|passwd|pwd|passphrase)\s*[:=]\s*["'][^"']{6,}["']`
- **시크릿 키**: `(secret_key|api_secret|client_secret|private_key|encryption_key)\s*[:=]\s*["'][^"']+["']`
- **프라이빗 키 블록**: `-----BEGIN (RSA |EC |DSA |OPENSSH |PGP )?(PRIVATE KEY|ENCRYPTED PRIVATE KEY)-----`
- **DB 연결 문자열**: `(postgres|postgresql|mongodb(\+srv)?|mysql|redis|amqp|mssql)://[^:\s]+:[^@\s]+@`
- **JDBC URL**: `jdbc:[a-z]+://[^:\s]+:[^@\s]+@`

##### 인프라 / 사설 네트워크

- 내부 도메인: `*.internal.`, `*.corp.`, `*.local`, `*.lan`, `*.intranet`
- 사설 IP: `10\.\d+\.\d+\.\d+`, `192\.168\.\d+\.\d+`, `172\.(1[6-9]|2\d|3[01])\.\d+\.\d+`
- staging/dev URL: `*.staging.`, `*-staging.`, `*-dev.`, `localhost:\d+` (포트번호 노출 시)

##### PII (개인정보)

| 카테고리 | 패턴 |
|---|---|
| 신용카드 | `\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b` (Luhn 알고리즘 추가 검증 권장) |
| 이메일 (사용자 데이터) | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` (단, git author email은 예외) |
| 한국 주민번호 | `\d{6}[-]?[1-4]\d{6}` (앞 6자리 + 뒷자리 첫 글자 1-4) |
| 한국 외국인등록번호 | `\d{6}[-]?[5-8]\d{6}` |
| 한국 전화번호 평문 | `01[0-9][-]?\d{3,4}[-]?\d{4}`, `\+82-?10-?\d{3,4}-?\d{4}` |
| 한국 운전면허 | `\d{2}-\d{2}-\d{6}-\d{2}` |
| 한국 여권 | `[A-Z]\d{8}` (컨텍스트와 결합 시) |
| 미국 SSN | `\d{3}-\d{2}-\d{4}` |

#### 4d. 프롬프트 인젝션 / 숨김 영역 검사 (Copilot 사고 교훈)

GitHub Copilot이 PR 본문의 **HTML 주석에 숨긴 텍스트**를 통해 AWS 키를 유출한 사건이 있음. 따라서 본문 작성 전에 다음도 검사:

- HTML 주석: `<!--.*-->` 안의 내용
- Markdown 링크의 hidden URL: `[text](url)`의 url 부분
- Image alt text: `![alt](url)`
- Zero-width 문자 (`​`, `‌`, `﻿` 등)
- Markdown reference link 정의: `[ref]: url`

→ 발견된 모든 영역의 텍스트도 secret 패턴 검사 적용.

#### 4e. False Positive 처리

다음은 통과 (단, 의심 가면 사용자 확인):

- **예시 파일**: `.env.example`, `.env.sample`, `*.template`, `*.example.*`, `*.sample.*`
- **placeholder 키워드**: `your-api-key`, `your-token-here`, `xxx`, `change-me`, `<your-...>`, `[your-...]`, `INSERT_KEY_HERE`, `replace-me`, `placeholder`
- **명백한 가짜 값**: `test_key_1234`, `dummy`, `fake-token`, `mock-`, `example-`, `sample-`
- **테스트 디렉토리 내**: `test/`, `tests/`, `__tests__/`, `spec/`, `e2e/`, `cypress/` 안의 발견은 confirmation 후 처리
- **주석 직후**: `# example`, `// placeholder`, `// fake`, `# fake` 같은 주석 직후 패턴
- **이미 `.gitignore`된 파일**: 실수로 staged된 게 아니면 통과
- **공개 정보**: Firebase API key (앱에 노출되는 정보), Mapbox public token, etc — 단, secret이 아닌 public이라는 것이 명확할 때만

#### 4f. 발견 시 처리

위반 발견 시 **본문 작성 즉시 중단** + 다음 형식으로 경고:

```
⚠️ 보안 검사에서 위험 패턴 발견 — PR 작성 중단

[위험 파일]
- .env (Production environment 변수 파일)
- src/keys/private.pem (프라이빗 키)

[Secret 패턴]
- src/config/auth.ts:42 — OpenAI API key (sk-...)
- src/db/connection.ts:18 — DB 연결 문자열 (postgres://user:pass@...)
- README.md:89 — Slack webhook URL

[PII]
- migrations/seed.sql:12 — 사용자 이메일 100건
- src/test/mock-user.ts:5 — 한국 주민번호 형식

[프롬프트 인젝션 의심]
- 커밋 메시지 안 HTML 주석에 숨겨진 토큰 패턴

이 정보들은 PR 본문에 절대 포함하지 않습니다.

⚠️ **그러나 git history에 그대로 박혀 있습니다.**

긴급 권고 (public repo면 더더욱):
1. 노출된 키는 **지금 즉시** 회전(rotate) — 봇이 수 분 내 수집함
2. 해당 secret을 사용하는 모든 서비스 monitoring 강화
3. secret을 환경변수/secret manager로 이동 (.env는 .gitignore에)
4. git history에서 제거: `git filter-repo --invert-paths --path <file>`
5. force push 전 팀에 공지 (collaborators 영향)
6. 회사 보안팀에 사고 보고 (PII 노출 시 필수)

진행 옵션:
[1] 작업 중단 → 정리 후 재시도 (강력 권장)
[2] 본문에서만 마스킹하고 PR 진행 (코드는 사용자가 별도 정리)
[3] False positive (예시/테스트라서 무시) — 위치 알려주면 화이트리스트
```

#### 4g. 본문 인용 시 마스킹 룰

옵션 [2]로 진행하더라도 본문에는 **절대 secret 값 인용 금지**:

- ✅ 위치만 표기: `src/config/auth.ts:42에 인증 키 추가`
- ✅ 마스킹: `<REDACTED:api-key>`, `<REDACTED:db-password>`, `<REDACTED:pii>`
- ❌ 부분 인용도 금지: `sk-abc...xyz` (앞뒤 일부도 키 식별/추론 가능)
- ❌ 해시 인용도 금지: SHA256 해시 결과도 무차별 대입에 사용될 수 있음

#### 4h. 도구 통합 권고

LLM 기반 단일 검사는 한계 있음. **반드시 자동화 도구와 함께 사용**:

- **gitleaks** — pre-commit hook 강력 권장 (150+ 패턴, 빠름)
- **TruffleHog** — CI/CD pipeline에 (700+ detector, 실제 API 검증)
- **GitHub Secret Scanning** — repo Settings에서 활성화 (Push protection 권장)
- **GitGuardian** — 다국어 + 실시간 monitoring

이 스킬은 **마지막 안전망**이지 1차 방어선이 아님. 사용자에게 알릴 것:
> 보안 검사를 통과했더라도, gitleaks/TruffleHog 같은 전용 도구로 다시 한 번 스캔하는 것을 권장합니다.

### Step 5: PR 제목 작성

규칙:
- **능동태, 현재형 동사**로 시작 ("Add" ✓, "Added" ✗, "Adding" ✗)
- 패턴: `<동사> <대상> [to/in/for <문맥>]`
- **영어 우선**. `language: ko` 인자가 있으면 한국어.
- 70자 이내

좋은 예: `Add user authentication with JWT` / `Fix race condition in queue worker`
나쁜 예: `Added auth` (과거형) / `Fixed bug` (모호함)

### Step 6: PR 본문 작성

#### 템플릿 우선순위

1. **레포 PR 템플릿 발견 시 그 구조 사용** (`PR Template (single)` 결과)
   - 템플릿의 섹션 헤더와 체크리스트 그대로 유지
   - 우리 룰 (능동태, 현재 상태 진술 등) 모두 적용
   - 빈 섹션은 통째로 제거 (Critical Rules)
2. **다중 템플릿** (`PULL_REQUEST_TEMPLATE/` 디렉토리)
   - 사용자에게 "어떤 템플릿 쓸까요? [목록]" 물음
3. **템플릿 없음** → 기본 구조 사용:

```markdown
## Why

이 변경이 왜 필요했나? 어떤 문제·요구사항·기회를 해결하는가?

## Approach

선택한 구현 방향과 그 이유.

## How it works

핵심 동작 방식과 주요 변경점:
- 새 모듈/함수/클래스
- 변경된 공개 인터페이스
- 추가된 의존성

## Links

- 참고 자료:
```

#### 이슈 자동 닫기 (`Closes #`)

`closes:#123` 인자가 있거나 분석 중 명확한 이슈 링크를 발견하면, **본문 첫 줄에** 키워드 사용:

```markdown
Closes #123
Fixes #456
Resolves #789

## Why
...
```

- 키워드: `Closes` / `Fixes` / `Resolves` 모두 머지 시 이슈 자동 닫힘
- 한 이슈당 한 줄
- 인자에 명시 안 됐고 분석상 모호하면 사용자에게 묻기: "이 PR로 닫을 이슈가 있나요? (예: #123)"

#### Co-author 푸터

Contributors에서 본인(`gh api user --jq .login`)을 제외한 인간 작성자가 1명 이상이면 본문 마지막에 추가:

```markdown
---

Co-authored-by: Alice <alice@example.com>
Co-authored-by: Bob <bob@example.com>
```

**제외 룰**:
- 본인은 제외 (Auto-Collected의 `GH user` 결과로 매칭)
- 이메일에 `bot`, `noreply`, `[bot]` 포함 → 제외
- 이름이 `Claude`, `GitHub`, `dependabot` 등 봇 → 제외
- AI 관련 어떤 표현도 추가 금지

#### 기존 PR 업데이트 모드

- **변경 로그가 아닌 "현재 상태 전체"를 반영**
- "이번에 X도 추가했음" 같은 incremental 표현 금지
- 처음부터 PR이 이 상태였던 것처럼 작성

### Step 7: 검토 + 실행

작성된 제목·본문을 사용자에게 보여주고 컨펌 요청.

#### 신규 PR 생성

기본 명령 (assignee `@me` 자동):
```bash
gh pr create \
  --title "..." \
  --body "..." \
  --base {base-branch} \
  --assignee @me
```

인자에 따라 플래그 조합:
| 인자 | 추가 플래그 |
|------|------------|
| `draft` | `--draft` |
| `reviewer:user1,user2,org/team` | `--reviewer user1,user2,org/team` |
| `label:bug,backend` | `--label "bug,backend"` |
| `assignee:none` | `--assignee` 제거 |
| `assignee:user1` | `--assignee user1` (자기 자신 대신) |

#### 기존 PR 업데이트

```bash
gh pr edit {number} --title "..." --body "..."
```

라벨·리뷰어·assignee 인자가 있으면 **add 전용** (기존 보존):
```bash
gh pr edit {number} \
  --add-label "bug,backend" \
  --add-reviewer "alice" \
  --add-assignee "@me"
```

## Critical Rules (Non-Negotiable)

### Security & Privacy (최우선)

- 🛑 **Step 4 Security Filter는 절대 스킵 금지** — diff/커밋/PR 제목/브랜치 이름/숨김 영역(HTML 주석/Markdown link)까지 전부 검사
- 🛑 **위험 파일 차단** — `.env`, `*.pem`, `*.key`, `id_rsa`, `*.kubeconfig`, `credentials.json`, `*.sql.dump`, `*.csv`(사용자 데이터) 같은 파일이 diff에 추가/변경됐으면 PR 작성 중단
- ❌ **민감 정보 본문 노출 금지** — API 키/토큰/비밀번호/프라이빗 키/DB 연결 문자열/Webhook URL 모두 절대 인용 금지 (위치만 표기 또는 `<REDACTED>` 마스킹)
- ❌ **부분 인용·해시 인용도 금지** — `sk-abc...xyz` (앞뒤 일부도 추론 가능), SHA256 해시도 brute-force 가능
- ❌ **내부 인프라 정보 노출 금지** — 사설 IP, staging endpoint, 내부 도메인(`*.internal.`, `*.corp.`, `*.local`), localhost:포트
- ❌ **PII(개인정보) 노출 금지** — 사용자 이메일·신용카드·주민번호·외국인등록번호·전화번호·운전면허·여권번호 등 (단, git commit author 이메일은 공개 정보이므로 co-author 푸터에 허용)
- ❌ **PR 제목·커밋 메시지에 secret 금지** — s1ngularity 사고처럼 title injection으로 secret 노출 가능
- ❌ **HTML 주석·Markdown 숨김 영역에 secret 금지** — Copilot 사고처럼 hidden comment 통한 노출 차단
- ⚠️ **public repo에서 secret 발견 시 사용자에게 즉시 rotate 강조** — 봇이 수 분 내 수집함

### Style & Content

- ❌ AI/Claude 관련 표현 금지 ("Generated with Claude" 등)
- ❌ `Co-Authored-By: Claude` 헤더 절대 금지 (인간 contributor만 허용)
- ❌ 이모지·아이콘 사용 금지 (사용자가 명시적 요청한 경우만)
- ❌ 변경 로그식 표현 금지 ("이번 PR에서는 X를 추가했습니다") → 현재 상태 진술로
- ❌ "더 좋게", "개선했다" 같은 모호한 표현 → 구체적 효과 명시 (수치, 시나리오)
- ❌ 빈 섹션 제출 금지 — 해당 없는 섹션은 통째로 생략

## Language Policy

- **PR 제목**: 영어 우선. `language: ko`로 한국어.
- **PR 본문**: 한국어 기본. `language: en`로 영어.
- 기술 용어는 영어 원문 그대로 (`lifetime`, `lazy loading`, `race condition` 등)

## Invocation

```
/pr-writer:write
/pr-writer:write develop
/pr-writer:write main language: en
/pr-writer:write main draft reviewer:alice,bob/team-frontend
/pr-writer:write main label:bug,backend closes:#142
/pr-writer:write main assignee:none
```
