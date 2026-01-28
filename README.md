# Self-Made Skills

[![License](https://img.shields.io/badge/License-MIT-da7756?style=flat)](./LICENSE)
[![Plugins](https://img.shields.io/badge/Plugins-1-da7756?style=flat)](#plugins)
[![Platform](https://img.shields.io/badge/Claude_Code-da7756?style=flat&logo=anthropic&logoColor=white)](https://code.claude.com)

직접 만든 Claude Code 플러그인 & 스킬 모음입니다.

<br>

## 🚀 Quick Start

### Claude Code에서 설치

터미널에서 아래 명령어를 실행하세요:

```bash
# 마켓플레이스 추가
claude plugin marketplace add gitjay3/self-made-skills

# 플러그인 설치 - User (기본): 내 모든 프로젝트에서 사용
claude plugin install troubleshoot-logger@self-made-skills

# 플러그인 설치 - Project: 팀 공유 (git에 포함됨)
claude plugin install troubleshoot-logger@self-made-skills --scope project

# 플러그인 설치 - Local: 이 프로젝트에서 나만 사용
claude plugin install troubleshoot-logger@self-made-skills --scope local

# 플러그인 삭제
claude plugin uninstall troubleshoot-logger@self-made-skills
```

### 설치 확인

- Claude Code에서 `/plugin` → Installed 탭
- 파일: `~/.claude/plugins/installed_plugins.json`

<br>

## 📦 Plugins

| 플러그인 | 설명 | 버전 |
|---------|------|------|
| [troubleshoot-logger](./plugins/troubleshoot-logger) | STAR 기법 + 근본 원인 분석으로 트러블슈팅 자동 기록 | `v1.0.0` |

<br>

## 📖 Details

### troubleshoot-logger

> 에러/버그 해결 후 트러블슈팅 과정을 STAR 기법으로 자동 문서화합니다.

**주요 기능**

- STAR 기법 (Situation, Task, Action, Result)
- 5 Whys 근본 원인 분석
- 재발 방지 체크리스트
- 태그 기반 분류
- `docs/troubleshooting/YYYY-MM-DD-title.md` 자동 저장

**자동 제안 조건**

Claude가 다음 문제를 해결하면 자동으로 기록을 제안합니다:

| 카테고리 | 상황 |
|---------|------|
| 에러 및 예외 | 컴파일/빌드 에러, 런타임 에러, 타입/문법 에러, 스택 트레이스 분석 |
| 테스트 및 검증 | 테스트 실패 수정, CI/CD 실패 해결, 린트/포맷 에러 |
| 환경 및 설정 | 의존성 충돌, 환경 변수, Docker, 권한/경로 문제 |
| 성능 및 동작 | 버그 수정, 무한 루프, 메모리 누수, API/DB 오류 |
| 통합 및 호환성 | 라이브러리 호환성, Git 충돌, 브라우저/플랫폼 이슈 |

**사용법**

```bash
# 자동: Claude가 문제 해결 후 기록 제안
# 수동:
/troubleshoot-logger:log
/troubleshoot-logger:log api-timeout-fix
```

<details>
<summary><b>출력 템플릿 보기</b></summary>

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

</details>

**태그 가이드**

| 카테고리 | 태그 |
|---------|------|
| 영역 | `#backend` `#frontend` `#database` `#infra` `#auth` |
| 유형 | `#bug` `#performance` `#security` `#config` |
| 난이도 | `#easy` `#medium` `#hard` |

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
