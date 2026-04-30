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
4. **제목 생성** — 능동태 현재형 동사
5. **본문 작성** — 레포 PR 템플릿 우선, 없으면 기본 구조
6. **검토 + 실행** — `gh pr create` 또는 `gh pr edit`

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

- AI/Claude 관련 표현 절대 금지 ("Generated with Claude" 등)
- `Co-Authored-By: Claude` 헤더 금지 (인간 contributor만 허용)
- 이모지·아이콘 금지 (명시 요청 시만)
- "이번 PR에서는 X를 추가했습니다" 같은 변경 로그식 표현 금지 — 현재 상태 진술로
- 빈 섹션 금지 — 해당 없으면 섹션 통째로 생략

<br>

## 언어 정책

- **제목**: 영어 우선. `language: ko` 인자로 한국어.
- **본문**: 한국어 기본. `language: en` 인자로 영어.
- 기술 용어는 영어 원문 그대로 (`lifetime`, `lazy loading`, `race condition` 등)

<br>

## 라이선스

MIT
