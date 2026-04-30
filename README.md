# Self-Made Skills

[![License](https://img.shields.io/badge/License-MIT-da7756?style=flat)](./LICENSE)
[![Plugins](https://img.shields.io/badge/Plugins-5-da7756?style=flat)](#plugins)
[![Platform](https://img.shields.io/badge/Claude_Code-da7756?style=flat&logo=anthropic&logoColor=white)](https://code.claude.com)

직접 만든 Claude Code 플러그인 & 스킬 모음입니다.

<br>

## 🚀 Quick Start

### Claude Code에서 설치

**Claude Code 내부에서 (슬래시 커맨드)**:

```bash
# 1. 마켓플레이스 추가 (한 번만)
/plugin marketplace add gitjay3/self-made-skills

# 2. 플러그인 설치
/plugin install troubleshoot-logger@self-made-skills

# 3. 활성화 (현재 세션에 즉시 반영)
/reload-plugins
```

**터미널 CLI에서 (스코프 지정 가능)**:

```bash
# User (기본): 내 모든 프로젝트에서 사용
claude plugin install troubleshoot-logger@self-made-skills

# Project: 팀 공유 (.claude/settings.json에 추가됨)
claude plugin install troubleshoot-logger@self-made-skills --scope project

# Local: 이 프로젝트에서 나만 사용
claude plugin install troubleshoot-logger@self-made-skills --scope local
```

**플러그인 삭제**:

```bash
/plugin uninstall troubleshoot-logger@self-made-skills
# 또는: claude plugin uninstall troubleshoot-logger@self-made-skills
```

### 설치 확인

- Claude Code에서 `/plugin` → Installed 탭
- 파일: `~/.claude/plugins/installed_plugins.json`

<br>

## 📦 Plugins

| 플러그인 | 설명 | 버전 |
|---------|------|------|
| [troubleshoot-logger](./plugins/troubleshoot-logger) | STAR 기법 + 근본 원인 분석으로 트러블슈팅 자동 기록 | `v1.0.0` |
| [learning-logger](./plugins/learning-logger) | 새로 배운 개념·API·문법·패턴을 TIL 형식으로 자동 기록 | `v1.0.0` |
| [decision-logger](./plugins/decision-logger) | 기술·아키텍처 의사결정을 ADR 형식으로 자동 기록 | `v1.0.0` |
| [pr-writer](./plugins/pr-writer) | 브랜치 git 컨텍스트로 GitHub PR 제목·본문 자동 작성 | `v1.0.0` |
| [skill-recommender](./plugins/skill-recommender) | 프로젝트·맥락 분석으로 마켓 내 적절한 플러그인 추천 (메타) | `v1.0.0` |

<br>

## 📖 Details

### troubleshoot-logger

> 에러/버그 해결 후 트러블슈팅 과정을 STAR 기법으로 자동 문서화합니다.

<br>

**주요 기능**

- STAR 기법 (Situation, Task, Action, Result)
- 5 Whys 근본 원인 분석
- 재발 방지 체크리스트
- 태그 기반 분류
- `docs/troubleshooting/YYYY-MM-DD-title.md` 자동 저장

<br>

**자동 제안 조건**

Claude가 다음 문제를 해결하면 자동으로 기록을 제안합니다:

| 카테고리 | 상황 |
|---------|------|
| 에러 및 예외 | 컴파일/빌드 에러, 런타임 에러, 타입/문법 에러, 스택 트레이스 분석 |
| 테스트 및 검증 | 테스트 실패 수정, CI/CD 실패 해결, 린트/포맷 에러 |
| 환경 및 설정 | 의존성 충돌, 환경 변수, Docker, 권한/경로 문제 |
| 성능 및 동작 | 버그 수정, 무한 루프, 메모리 누수, API/DB 오류 |
| 통합 및 호환성 | 라이브러리 호환성, Git 충돌, 브라우저/플랫폼 이슈 |

<br>

**사용법**

```bash
# 자동: Claude가 문제 해결 후 기록 제안

# 수동: 자동 제안이 안 뜨거나, 직접 기록하고 싶을 때
스킬 써서 트러블슈팅 내용 정리해줘
스킬 써서 방금 해결한 버그 기록해줘
```

<br>

### learning-logger

> 새로 배운 개념·API·문법·패턴을 TIL(Today I Learned) 형식으로 자동 문서화합니다.

<br>

**주요 기능**

- TL;DR / 무엇을 / 왜 / 어떻게 / 주의할 점 / 관련 자료 구조
- 한 토픽당 한 파일 (검색·재참조 쉬움)
- 태그 기반 분류 (언어/영역/유형/난이도)
- `docs/til/YYYY-MM-DD-topic.md` 자동 저장

<br>

**자동 제안 조건**

Claude가 다음 내용을 사용자가 처음 접하는 형태로 설명하면 자동으로 기록을 제안합니다:

| 카테고리 | 상황 |
|---------|------|
| 언어·문법 | 새 키워드/연산자/제네릭/lifetime/decorator 등 처음 다룬 경우 |
| 라이브러리·API | 처음 쓰는 라이브러리 사용법, 익숙한 API의 새 옵션 |
| 디자인 패턴·아키텍처 | Observer/Strategy/CQRS/Event Sourcing 등 처음 적용 |
| 도구·명령어 | 새 CLI 명령, IDE 단축키, 빌드/테스트 도구 옵션 |
| 개념·이론 | 컴퓨터 과학·웹 표준·프로토콜 등 이론 학습 |

<br>

**사용법**

```bash
# 자동: Claude가 새로운 내용 설명 후 기록 제안

# 수동: 자동 제안이 안 뜨거나, 직접 기록하고 싶을 때
스킬 써서 방금 배운 내용 TIL로 기록해줘
스킬 써서 오늘 배운 거 정리해줘
```

<br>

### decision-logger

> 기술·아키텍처 의사결정을 ADR(Architecture Decision Record) 형식으로 자동 문서화합니다.

<br>

**주요 기능**

- MADR 표준 기반 구조 (Context / Decision / Alternatives / Consequences)
- 시퀀스 번호 자동 부여 (`NNNN-title.md`)
- 검토 대안과 기각 사유를 표로 정리
- 상태 라이프사이클 (Proposed → Accepted → Deprecated / Superseded)
- `docs/adr/` 폴더에 저장 + index README 자동 유지

<br>

**자동 제안 조건**

Claude가 다음 의사결정을 감지하면 자동으로 기록을 제안합니다:

| 카테고리 | 상황 |
|---------|------|
| 라이브러리·프레임워크 | 같은 영역 라이브러리 비교 후 채택, 라이브러리 교체 결정 |
| 데이터·저장소 | DB 종류 선택, ORM 채택, 캐싱·검색·메시지큐 결정 |
| 아키텍처·패턴 | monolith vs microservice, Event Sourcing/CQRS 적용 |
| 인프라·플랫폼 | 클라우드 제공자, 컨테이너 오케스트레이션, CI/CD 도구 변경 |
| 개발 도구·컨벤션 | 패키지 매니저·빌드 도구 변경, 스타일·테스트 표준화 |

<br>

**사용법**

```bash
# 자동: Claude가 결정 직후 기록 제안

# 수동: 자동 제안이 안 뜨거나, 직접 기록하고 싶을 때
스킬 써서 방금 결정한 내용 ADR로 기록해줘
스킬 써서 우리가 X 선택한 거 정리해줘
```

<br>

### pr-writer

> 브랜치의 git 컨텍스트를 분석해 구조화된 GitHub PR 제목·본문을 자동 작성합니다.

<br>

**주요 기능**

- 브랜치 전체 커밋·diff 종합 분석 (마지막 커밋이 아닌 누적 결과)
- 능동태 현재형 동사 PR 제목 (영어 기본, 한국어 옵션)
- 한국어 PR 본문 (Why / Approach / How it works / Test plan / Links)
- 신규 PR + 기존 PR 갱신 모드 자동 분기
- AI/Claude 표현·`Co-Authored-By` 자동 배제

<br>

**요구사항**

GitHub CLI(`gh`)가 설치·인증되어 있어야 합니다 (`gh auth login`).

<br>

**사용법**

```bash
# 기본 (base: main, 본문 한국어)
/pr-writer:write

# 다른 base 브랜치 / 본문 영어
/pr-writer:write develop
/pr-writer:write main language: en

# 자연어 호출
PR 올려줘
PR 본문 써줘
풀리퀘 만들어줘
```

<br>

### skill-recommender

> 프로젝트와 대화 맥락을 분석해 이 마켓 내 어떤 플러그인을 쓰면 좋을지 추천하는 메타 플러그인입니다.

<br>

**주요 기능**

- 프로젝트 분석 (`package.json`, `tsconfig.json`, `pyproject.toml`, `go.mod`, `.git/config` 등)
- 사용자 의도 추출 (인자 또는 최근 대화)
- 설치된 플러그인 자동 감지
- 카탈로그 기반 추천 (High / Medium / Low 적합도 + 이유)
- 자동 매칭·자동 설치 없음 — 명시적 호출 + 명시적 동의

<br>

**사용법**

```bash
# 일반 분석 + 추천
/skill-recommender:recommend

# 작업 의도 명시
/skill-recommender:recommend "PR 올리는 거 자동화하고 싶어"

# 자연어 호출
이 프로젝트에 어떤 도구가 좋을까?
self-made-skills에서 뭐 쓰면 좋아?
```

<br>

## 🤝 Contributing

1. Fork
2. Branch (`git checkout -b feature/new-skill`)
3. Commit (`git commit -m '새 스킬 추가'`)
4. Push (`git push origin feature/new-skill`)
5. Pull Request

<br>

## 📄 License

[MIT](./LICENSE)
