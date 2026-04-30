# Skill Recommender

프로젝트 파일과 대화 맥락을 분석해 `gitjay3/self-made-skills` 마켓의 적절한 플러그인을 추천하는 메타 플러그인입니다.

## 기능

- 프로젝트 분석 (`package.json`, `tsconfig.json`, `pyproject.toml`, `go.mod`, `.git/config` 등)
- 사용자 의도 추출 (인자 또는 최근 대화)
- 설치된 플러그인 자동 감지
- 카탈로그 기반 추천 (High / Medium / Low 적합도)
- 분석 결과 항상 표시 (추천이 없어도)
- 자동 설치·자동 매칭 없음 — 명시적 호출 + 명시적 동의 필요

<br>

## 설치

### 마켓플레이스에서 설치

```bash
/plugin marketplace add gitjay3/self-made-skills
/plugin install skill-recommender@self-made-skills
```

<br>

## 사용법

```bash
# 일반 분석 + 추천
/skill-recommender:recommend

# 작업 의도를 명시
/skill-recommender:recommend "PR 올리는 거 자동화하고 싶어"
/skill-recommender:recommend "이 프로젝트에 뭐 쓰면 좋을까"
```

또는 자연어 요청에도 반응합니다:
```
이 프로젝트에 어떤 도구가 좋을까?
self-made-skills에서 뭐 쓰면 좋아?
어떤 플러그인 깔면 도움될까?
```

<br>

## 출력 예시

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

<br>

## 카탈로그

추천 대상은 `gitjay3/self-made-skills` 마켓 내부 플러그인으로 한정됩니다.

| 플러그인 | 용도 |
|---------|------|
| troubleshoot-logger | 에러·버그 해결 과정 (STAR + 5Whys) 기록 |
| learning-logger | 새로 배운 내용 TIL 기록 |
| decision-logger | 기술·아키텍처 결정 ADR 기록 |
| pr-writer | GitHub PR 제목·본문 자동 작성 |

> 새 플러그인이 마켓에 추가되면 [`SKILL.md`](./skills/recommend/SKILL.md)의 카탈로그를 업데이트해야 합니다.

<br>

## 동작 원칙

- ❌ **자동 매칭 금지** — 사용자가 명시적으로 추천을 요청한 경우만 발화
- ❌ **자동 설치 금지** — 항상 명시적 명령 안내만
- ❌ **외부 플러그인 추천 금지** — 마켓 내부만
- ❌ **거절한 도구 재제안 금지** — 같은 세션에서 다시 권하지 않음
- ✅ **분석 결과 항상 표시** — 추천이 없어도 컨텍스트는 보여줌

<br>

## 라이선스

MIT
