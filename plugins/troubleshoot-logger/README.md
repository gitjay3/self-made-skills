# Troubleshoot Logger

STAR 기법 + 근본 원인 분석으로 트러블슈팅 과정을 자동 기록하는 Claude Code 플러그인입니다.

## 기능

- 에러/버그 해결 후 자동으로 기록 제안
- STAR (Situation, Task, Action, Result) 형식
- 5 Whys 근본 원인 분석
- 재발 방지 체크리스트
- 관련 파일/커밋 링크
- 태그 기반 분류
- `docs/troubleshooting/` 폴더에 날짜별 파일로 저장

<br>

## 설치

### 마켓플레이스에서 설치

```bash
/plugin marketplace add gitjay3/self-made-skills
/plugin install troubleshoot-logger@self-made-skills
```

<br>


## 사용법

### 자동 제안

Claude가 다음 상황에서 문제를 **성공적으로 해결**하면 자동으로 기록을 제안합니다:

**에러 및 예외 처리**
- 컴파일/빌드 에러를 수정한 경우
- 런타임 에러나 예외를 해결한 경우
- 타입 에러, 문법 에러를 고친 경우
- 스택 트레이스를 분석하고 원인을 해결한 경우

**테스트 및 검증**
- 실패하던 테스트를 통과시킨 경우
- CI/CD 파이프라인 실패를 해결한 경우
- 린트/포맷 에러를 수정한 경우

**환경 및 설정**
- 의존성 충돌이나 버전 문제를 해결한 경우
- 환경 변수, 설정 파일 문제를 고친 경우
- Docker, 컨테이너 관련 문제를 해결한 경우
- 권한, 경로 문제를 수정한 경우

**성능 및 동작**
- 버그로 인한 잘못된 동작을 수정한 경우
- 무한 루프, 메모리 누수를 해결한 경우
- API 호출 실패, 네트워크 문제를 고친 경우
- 데이터베이스 쿼리 오류를 수정한 경우

**통합 및 호환성**
- 라이브러리/프레임워크 호환성 문제를 해결한 경우
- Git 충돌, 병합 문제를 해결한 경우
- 브라우저/플랫폼 호환성 이슈를 고친 경우

### 수동 호출

```bash
/troubleshoot-logger:log
/troubleshoot-logger:log api-timeout-fix
```

<br>

## 출력 예시

`docs/troubleshooting/2026-01-28-api-latency-fix.md`:

````markdown
# API 응답 지연 문제 해결

> 기록일: 2026-01-28
> 태그: #backend #performance #database

## Situation (상황)

프로덕션 환경에서 API 응답이 5초 이상 걸리는 현상 발생
- 환경: Production (AWS ECS)
- 증상: GET /api/users 엔드포인트 타임아웃
- 에러: `TimeoutError: Request timed out after 5000ms`

## Task (과제)

- API 응답 시간을 500ms 이하로 줄여야 함
- 서비스 중단 없이 해결해야 함

## Action (행동)

1. 쿼리 실행 계획 분석
   ```sql
   EXPLAIN ANALYZE SELECT * FROM users WHERE ...
   ```

2. N+1 쿼리 문제 발견 → eager loading 적용
   ```python
   # Before
   users = User.query.all()

   # After
   users = User.query.options(joinedload(User.profile)).all()
   ```

3. Redis 캐싱 레이어 추가

## Result (결과)

- 응답 시간: 5000ms → 200ms (96% 개선)
- 서버 부하: 40% 감소

## Root Cause (근본 원인 분석)

1. **Why**: 왜 API가 느렸나? → DB 쿼리가 오래 걸림
2. **Why**: 왜 쿼리가 오래 걸렸나? → N+1 문제 + 인덱스 없음
3. **Why**: 왜 N+1 문제가 있었나? → ORM 기본 설정 사용, 코드 리뷰 시 누락

## Prevention (재발 방지)

- [ ] 쿼리 성능 모니터링 대시보드 추가
- [ ] 코드 리뷰 시 ORM 쿼리 확인 체크리스트 추가
- [x] 개발 환경에 쿼리 로깅 활성화

## Related (관련 자료)

- 파일: `src/api/users.py`, `src/cache/user_cache.py`
- 커밋: `a1b2c3d`
- PR: #456
````

<br>

## 태그 가이드

| 카테고리 | 태그 예시 |
|---------|----------|
| 영역 | `#backend` `#frontend` `#database` `#infra` `#auth` |
| 유형 | `#bug` `#performance` `#security` `#config` |
| 난이도 | `#easy` `#medium` `#hard` |

<br>

## 라이선스

MIT
