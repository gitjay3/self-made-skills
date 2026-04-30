# PR Writer

브랜치의 git 컨텍스트를 분석해 구조화된 GitHub PR 제목·본문을 자동 작성하는 Claude Code 플러그인입니다.

## 기능

- 브랜치 전체 커밋·diff를 종합 분석 (마지막 커밋이 아닌 누적 결과)
- 능동태 현재형 동사 PR 제목 (영어 기본, 한국어 옵션)
- 한국어 PR 본문 (Why / Approach / How it works / Links)
- 신규 PR 생성 + 기존 PR 본문 갱신 모드 자동 분기
- 푸시 안 된 커밋 자동 감지·안내
- AI/Claude 표현·`Co-Authored-By` 자동 배제

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

### 명시적 호출

```bash
# 기본 (base: main, 본문 한국어)
/pr-writer:write

# 다른 base 브랜치
/pr-writer:write develop

# 본문도 영어로
/pr-writer:write main language: en
```

### 자동 매칭

다음과 같은 자연어 요청에도 반응합니다:

```
PR 올려줘
PR 본문 써줘
풀리퀘 만들어줘
```

기존 PR이 이미 열려있다면 **업데이트 모드**로 전환되어 본문을 현재 상태에 맞게 다시 작성합니다.

<br>

## 출력 예시

### PR 제목

```
Add user authentication with JWT
```

### PR 본문

````markdown
## Why

기존 세션 기반 인증은 모바일 클라이언트와 외부 API 통합에서 한계가 있었음.
관련 이슈: #142

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

- 관련 이슈: #142
- 참고: https://datatracker.ietf.org/doc/html/rfc7519
````

<br>

## 핵심 룰 (위반 금지)

- AI/Claude 관련 표현 절대 금지 ("Generated with Claude" 등)
- `Co-Authored-By: Claude` 헤더 금지
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
