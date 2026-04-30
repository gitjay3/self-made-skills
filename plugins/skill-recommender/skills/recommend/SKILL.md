---
name: recommend
description: This skill should be used when the user asks "어떤 도구 쓰면 좋아", "이 프로젝트에 맞는 도구 추천해줘", "self-made-skills에서 뭐 쓰면 좋아", "어떤 플러그인 깔면 도움될까", "지금 상황에 맞는 도구 추천", "what tools should I use", "recommend a skill for this project", "which skill fits this task", or wants an overview of available tools in the gitjay3/self-made-skills marketplace. Analyzes project files (package.json/tsconfig.json/pyproject.toml/etc) and conversation context, then recommends matching plugins from the marketplace catalog with High/Medium/Low fit scores and explicit reasoning.
argument-hint: "[task description (optional)]"
---

# Skill Recommender

Analyze project context + user intent, then recommend which `gitjay3/self-made-skills` plugins to use.

## Marketplace Catalog

The plugins below are the **only** ones available in `gitjay3/self-made-skills`. When recommending, do not suggest external plugins.

| Plugin | Purpose | When to Use | Invocation |
|--------|---------|-------------|------------|
| **troubleshoot-logger** | 에러·버그 해결 과정을 STAR + 5Whys로 문서화 | 트러블슈팅 후 재발 방지·회고가 필요할 때 | `/troubleshoot-logger:log` |
| **learning-logger** | 새로 배운 개념·API·문법·패턴을 TIL로 기록 | 학습 노트 필요, 솔로 개발자, 신규 도메인 학습 중 | `/learning-logger:log` |
| **decision-logger** | 기술·아키텍처 결정을 ADR로 문서화 | 의미 있는 라이브러리·DB·패턴 결정이 발생할 때 | `/decision-logger:log` |
| **pr-writer** | 브랜치 git 컨텍스트로 GitHub PR 제목·본문 자동 작성 | GitHub 워크플로우 사용, 정기적 PR 작성 | `/pr-writer:write` |
| **skill-recommender** | (이 도구 자신 — 추천 결과에서 제외) | — | — |

> **Maintenance note**: 새 플러그인이 마켓에 추가되면 이 카탈로그도 업데이트해야 합니다.

## Workflow (5 Steps)

### Step 1: 컨텍스트 수집

Run in parallel:

**1a. 사용자 의도 분석**:
- 인자가 있으면: 작업 설명에서 키워드 추출 (예: "PR 올리는 거 도와줘" → PR 영역)
- 인자가 없으면: 최근 대화 히스토리에서 의도 추론. 추론 불가능하면 "명시되지 않음"으로 처리

**1b. 프로젝트 파일 읽기** (Glob + Read, 존재하는 것만):
- `package.json`, `tsconfig.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, `docker-compose.yml`
- `.git/config`로 원격 호스팅(GitHub/GitLab) 확인
- `docs/` 디렉토리 구조 — 이미 어떤 문서를 쓰고 있는지

**1c. 설치된 플러그인 확인**:
- `~/.claude/plugins/installed_plugins.json` Read 시도
- 마켓 카탈로그의 어떤 플러그인이 이미 설치되어 있는지 파악
- 파일이 없거나 읽기 실패하면 "확인 불가"로 처리하고 진행

### Step 2: 카탈로그 매칭

For each plugin in the marketplace catalog (skill-recommender 자기 자신 제외), score relevance:

| Score | 기준 |
|-------|------|
| **High** | 사용자 의도와 직접 일치, 또는 프로젝트 컨텍스트가 강하게 매칭 |
| **Medium** | 잠재적으로 유용하지만 사용자가 직접 요청하진 않음 |
| **Low** | 현재 컨텍스트와 무관 |

매칭 예시:
- "PR 올려줘" → pr-writer (High)
- 사용자가 새 라이브러리 학습 중 → learning-logger (High)
- GitHub 사용 + 활발한 커밋 → pr-writer (Medium)
- 빈 프로젝트 / 단일 파일 편집 → 모두 Low

### Step 3: 추천 표시 (ALWAYS)

분석 결과는 **항상** 사용자에게 보여줍니다 — 추천이 없어도 그 사실을 명시.

추천이 있을 때:

```
## 분석 결과

**감지된 컨텍스트**:
- 스택: Node.js, TypeScript, React
- Git 호스팅: GitHub
- 사용자 의도: 새 인증 라이브러리 학습 중

**이미 설치된 도구**: troubleshoot-logger

**추천 도구**:

| # | 도구 | 적합도 | 이유 |
|---|------|--------|------|
| 1 | learning-logger | High | 새 라이브러리 학습 중 — TIL로 정리하면 효과적 |
| 2 | decision-logger | Medium | 인증 라이브러리 선택은 ADR 가치가 큼 |
| 3 | pr-writer | Medium | GitHub 사용 프로젝트, PR 작성 시 도움 |

추천 도구 번호를 선택하세요 (여러 개: 1,2 / 건너뛰기: skip)
```

추천이 없을 때 (모두 Low):

```
## 분석 결과

**감지된 컨텍스트**: ...

현재 컨텍스트엔 특별히 추천할 도구가 없습니다. 직접 사용해보고 싶은 도구가 있으면 알려주세요. 사용 가능한 도구 목록:
- troubleshoot-logger / learning-logger / decision-logger / pr-writer
```

### Step 4: 사용자 선택 처리

선택된 도구별로 분기:

**이미 설치된 경우** — 호출 명령만 안내:
> troubleshoot-logger는 이미 설치되어 있어요. `/troubleshoot-logger:log`로 바로 사용 가능합니다.

**설치되지 않은 경우** — 명시적 설치 명령 안내 (자동 설치 X):
> learning-logger를 설치하려면:
> ```
> /plugin install learning-logger@self-made-skills
> ```

**`skip` / `건너뛰기` / `pass`** — 그대로 종료. 이 세션에서 다시 추천하지 않음.

### Step 5: 실행 컨텍스트 인계 (선택)

사용자가 선택한 도구를 즉시 사용하고 싶다고 표현하면 (예: "그럼 바로 써봐", "지금 정리해줘"), 해당 슬래시 커맨드를 호출하여 컨텍스트를 인계합니다.

예: pr-writer 추천 + 사용자 동의 → `/pr-writer:write` 호출

## UX Rules (Non-Negotiable)

- ❌ **자동 매칭 모드 금지** — 사용자가 명시적으로 추천을 요청한 경우만 발화
- ❌ **Max 1 recommendation set per turn** — 한 턴에 한 번만 보여주고 끝
- ❌ **자동 설치 금지** — 항상 명시적 명령 안내만
- ❌ **사소한 작업에 추천 X** — 단일 파일 편집, 간단한 질문 등
- ❌ **거절한 도구 재제안 금지** — 같은 세션에서 다시 권하지 않음
- ❌ **외부 플러그인 추천 금지** — 위 카탈로그 외 추천 안 함
- ✅ **다 커버되어도 분석 결과는 항상 표시**
- ✅ **사용자가 사용한 언어 그대로 응답** (한국어 → 한국어)

## Invocation

```
/skill-recommender:recommend
/skill-recommender:recommend "PR 올리는 거 자동화하고 싶어"
/skill-recommender:recommend "이 프로젝트에 뭐 쓰면 좋을까"
```
