# Self-Made Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Plugins](https://img.shields.io/badge/Plugins-1-brightgreen.svg)](#-플러그인-목록)
[![Platform](https://img.shields.io/badge/Platform-Claude%20Code-blue.svg)](https://code.claude.com)

직접 만든 Claude Code 플러그인 & 스킬 모음입니다.

<br>

## 📑 목차

- [Self-Made Skills](#self-made-skills)
  - [📑 목차](#-목차)
  - [⚡ 빠른 시작](#-빠른-시작)
    - [Claude Code에서 설치](#claude-code에서-설치)
    - [로컬 테스트](#로컬-테스트)
  - [📦 플러그인 목록](#-플러그인-목록)
  - [🔧 상세 설명](#-상세-설명)
    - [troubleshoot-logger](#troubleshoot-logger)
      - [✨ 주요 기능](#-주요-기능)
      - [📝 사용법](#-사용법)
      - [📋 출력 템플릿](#-출력-템플릿)
      - [🏷️ 태그 가이드](#️-태그-가이드)
  - [📁 프로젝트 구조](#-프로젝트-구조)
  - [🤝 기여하기](#-기여하기)
  - [📄 라이선스](#-라이선스)

<br>

## ⚡ 빠른 시작

### Claude Code에서 설치

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add gitjay3/self-made-skills

# 2. 플러그인 설치
/plugin install troubleshoot-logger@self-made-skills
```

### 로컬 테스트

```bash
git clone https://github.com/gitjay3/self-made-skills.git
cd self-made-skills
claude --plugin-dir ./plugins/troubleshoot-logger
```

<br>

## 📦 플러그인 목록

| 플러그인 | 설명 | 버전 | 상태 |
|---------|------|------|------|
| [troubleshoot-logger](./plugins/troubleshoot-logger) | STAR 기법 + 근본 원인 분석으로 트러블슈팅 자동 기록 | v1.0.0 | ✅ 사용 가능 |

<br>

## 🔧 상세 설명

### troubleshoot-logger

> 에러/버그 해결 후 트러블슈팅 과정을 STAR 기법으로 자동 문서화합니다.

#### ✨ 주요 기능

- **STAR 기법**: Situation, Task, Action, Result 구조화
- **5 Whys 분석**: 근본 원인까지 파고드는 분석
- **재발 방지**: 체크리스트로 후속 조치 관리
- **태그 시스템**: `#backend` `#performance` 등 분류
- **자동 저장**: `docs/troubleshooting/YYYY-MM-DD-title.md`

#### 📝 사용법

```bash
# 자동 호출: Claude가 문제 해결 후 기록 제안
# 수동 호출:
/troubleshoot-logger:log
/troubleshoot-logger:log api-timeout-fix
```

#### 📋 출력 템플릿

```markdown
# API 응답 지연 문제 해결

> 기록일: 2026-01-28
> 태그: #backend #performance #database

## Situation (상황)
프로덕션 환경에서 API 응답이 5초 이상 걸리는 현상 발생...

## Task (과제)
API 응답 시간을 500ms 이하로 줄여야 함...

## Action (행동)
1. 쿼리 실행 계획 분석
2. N+1 쿼리 문제 발견 → eager loading 적용
3. Redis 캐싱 레이어 추가

## Result (결과)
- 응답 시간: 5000ms → 200ms (96% 개선)
- 서버 부하: 40% 감소

## Root Cause (근본 원인 분석)
1. Why: 왜 API가 느렸나? → DB 쿼리가 오래 걸림
2. Why: 왜 쿼리가 오래 걸렸나? → N+1 문제 + 인덱스 없음
3. Why: 왜 N+1 문제가 있었나? → ORM 기본 설정 사용

## Prevention (재발 방지)
- [ ] 쿼리 성능 모니터링 추가
- [ ] 코드 리뷰 시 쿼리 플랜 확인

## Related (관련 자료)
- 파일: `src/api/users.py`
- 커밋: `a1b2c3d`
```

#### 🏷️ 태그 가이드

| 카테고리 | 태그 |
|---------|------|
| 영역 | `#backend` `#frontend` `#database` `#infra` `#auth` |
| 유형 | `#bug` `#performance` `#security` `#config` |
| 난이도 | `#easy` `#medium` `#hard` |

<br>

## 📁 프로젝트 구조

```
self-made-skills/
├── .claude-plugin/
│   └── marketplace.json       # 마켓플레이스 정의
├── plugins/
│   └── troubleshoot-logger/
│       ├── .claude-plugin/
│       │   └── plugin.json    # 플러그인 메타데이터
│       ├── skills/
│       │   └── log/
│       │       └── SKILL.md   # 스킬 정의
│       └── README.md
├── LICENSE
└── README.md
```

<br>

## 🤝 기여하기

1. 이 저장소를 Fork 합니다
2. 새 브랜치를 생성합니다 (`git checkout -b feature/new-skill`)
3. 변경사항을 커밋합니다 (`git commit -m '새 스킬 추가'`)
4. 브랜치에 Push 합니다 (`git push origin feature/new-skill`)
5. Pull Request를 생성합니다

<br>

## 📄 라이선스

이 프로젝트는 [MIT 라이선스](./LICENSE)를 따릅니다.
