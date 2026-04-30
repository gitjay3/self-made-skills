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
- **GH user**: !`gh api user --jq .login 2>&1 || echo "UNKNOWN"`
- **Open PR**: !`gh pr view --json number,title,body,headRefName,baseRefName 2>&1 || echo "NO_OPEN_PR"`
- **PR Template (single)**: !`cat .github/pull_request_template.md 2>/dev/null || cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null || cat docs/pull_request_template.md 2>/dev/null || echo "NO_TEMPLATE"`
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

### Step 4: PR 제목 작성

규칙:
- **능동태, 현재형 동사**로 시작 ("Add" ✓, "Added" ✗, "Adding" ✗)
- 패턴: `<동사> <대상> [to/in/for <문맥>]`
- **영어 우선**. `language: ko` 인자가 있으면 한국어.
- 70자 이내

좋은 예: `Add user authentication with JWT` / `Fix race condition in queue worker`
나쁜 예: `Added auth` (과거형) / `Fixed bug` (모호함)

### Step 5: PR 본문 작성

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

### Step 6: 검토 + 실행

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
