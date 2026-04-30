---
name: write
description: This skill should be used when the user asks to "PR 올려줘", "PR 본문 써줘", "풀리퀘 만들어줘", "PR 만들어", "PR 갱신", "PR 본문 갱신", "create a PR", "open a pull request", "update PR description", "rewrite PR body", or wants to push a feature branch and open/update a GitHub pull request. Generates PR title (active voice, present-tense verb) and body (Why/Approach/How it works/Links sections) from the branch's full git context using gh CLI. Auto-detects existing PR for update mode.
argument-hint: "[base-branch (default: main)] [language: ko|en]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git branch:*), Bash(git log:*), Bash(git push:*), Bash(gh auth status:*), Bash(gh pr view:*), Bash(gh pr create:*), Bash(gh pr edit:*)
---

# PR Writer

Create or update a GitHub PR with a structured title and body, generated from the current branch's git context.

## Auto-Collected Context

The following context is gathered automatically before this skill runs:

- **Status**: !`git status --short`
- **Branch**: !`git branch --show-current`
- **Recent commits**: !`git log --oneline -20`
- **GH auth**: !`gh auth status 2>&1 || echo "NOT_AUTHENTICATED"`
- **Open PR (if any)**: !`gh pr view --json number,title,body,headRefName,baseRefName 2>&1 || echo "NO_OPEN_PR"`

## Arguments

User passed: `$ARGUMENTS`

Parse arguments from the string above:
- **base-branch**: first token that looks like a branch name (e.g. `main`, `master`, `develop`). Default: `main`.
- **language**: pattern `language: ko` or `language: en`. Default: `ko`.

## Workflow

### Step 1: Pre-flight Check

- If GH auth is `NOT_AUTHENTICATED`, instruct user to run `gh auth login` and stop.
- Check for unpushed commits: if `git status` or `git log` shows commits ahead of origin, prompt user:
  > 푸시되지 않은 커밋이 있어요. `git push` 먼저 진행할까요?
- If PR exists (Open PR section is not `NO_OPEN_PR`), switch to **update mode**.

### Step 2: Diff Against Base

Run `git diff {base-branch}...HEAD` via Bash to get the full diff against merge-base. Use the parsed `base-branch` from arguments.

### Step 3: 변경 분석

전체 커밋과 diff를 종합하여 식별:

1. **What** — 무엇이 추가·변경·삭제되었나 (단일 커밋이 아닌 **누적 결과**)
2. **Why** — 왜 변경되었나 (이슈 링크, 사용자 의도, 대화 맥락)
3. **How** — 핵심 구현 방식 (주요 알고리즘, 패턴, 새 의존성)
4. **Links** — 관련 이슈, 이전 PR, 참고 문서

> **중요**: 마지막 커밋에만 의존하지 말 것. 모든 커밋의 누적 결과를 분석.

### Step 4: PR 제목 작성

규칙:
- **능동태, 현재형 동사**로 시작 ("Add" ✓, "Added" ✗, "Adding" ✗)
- 패턴: `<동사> <대상> [to/in/for <문맥>]`
- **영어 제목 우선** (오픈소스·국제 협업 호환). 사용자가 `language: ko` 인자를 주면 한국어.
- 70자 이내

좋은 예:
- `Add user authentication with JWT`
- `Fix race condition in queue worker`
- `Refactor payment service to async`

나쁜 예:
- `Added auth` (과거형)
- `Fixed bug` (모호함)
- `유저 인증 추가함` (한국어 + 과거형)

### Step 5: PR 본문 작성

Default language: **한국어** (`language: en` 인자로 영어 전환).

Template:

```markdown
## Why

이 변경이 왜 필요했나? 어떤 문제·요구사항·기회를 해결하는가?
(관련 이슈가 있다면 여기 참조)

## Approach

선택한 구현 방향과 그 이유.
다른 접근을 검토했다면 간단히 언급.

## How it works

핵심 동작 방식과 주요 변경점:
- 새 모듈/함수/클래스
- 변경된 공개 인터페이스
- 추가된 의존성
- 데이터 흐름·시퀀스

## Links

- 관련 이슈: #
- 이전 PR:
- 참고 자료:
```

**기존 PR 업데이트 모드 (Step 1에서 PR 발견된 경우)**:
- **변경 로그가 아닌 "현재 상태 전체"를 반영**
- "이번에 X도 추가했음" 같은 incremental 표현 금지
- 처음부터 PR이 이 상태였던 것처럼 작성

### Step 6: 검토 + 실행

작성된 제목·본문을 사용자에게 보여주고 컨펌 요청.

컨펌 후 실행:
- **신규 PR**: `gh pr create --title "..." --body "..." --base {base-branch}`
- **기존 PR 업데이트**: `gh pr edit {number} --title "..." --body "..."`

## Critical Rules (Non-Negotiable)

다음 룰은 절대 위반 금지:

- ❌ AI/Claude 관련 표현 금지 ("Generated with Claude", "AI-generated" 등)
- ❌ `Co-Authored-By: Claude` 헤더 추가 금지
- ❌ 이모지·아이콘 사용 금지 (사용자가 명시적 요청한 경우만)
- ❌ 변경 로그식 표현 금지 ("이번 PR에서는 X를 추가했습니다") → 현재 상태 진술로
- ❌ "더 좋게", "개선했다" 같은 모호한 표현 → 구체적 효과 명시 (수치, 시나리오)
- ❌ 빈 섹션 제출 금지 — 해당 없는 섹션은 통째로 생략 (`Links`에 내용 없으면 섹션 자체 제거)

## Language Policy

- **PR 제목**: 영어 우선. 한국어 사내 프로젝트라면 `language: ko`.
- **PR 본문**: 한국어 기본. `language: en` 인자로 영어 전환.
- 기술 용어는 영어 원문 그대로 (예: "lifetime", "lazy loading", "race condition")

## Invocation

```
/pr-writer:write
/pr-writer:write develop
/pr-writer:write main language: en
```
