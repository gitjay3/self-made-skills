# Decision Logger

기술적·아키텍처 의사결정을 ADR(Architecture Decision Record) 형식으로 자동 기록하는 Claude Code 플러그인입니다.

## 기능

- 의미 있는 기술 결정 직후 자동으로 기록 제안
- MADR 표준 기반 구조 (Context / Decision / Alternatives / Consequences)
- 시퀀스 번호 자동 부여 (`NNNN-title.md`)
- 검토한 대안과 기각 사유를 표로 정리
- 상태 라이프사이클 (Proposed → Accepted → Deprecated / Superseded)
- `docs/adr/` 폴더에 저장 + index README 자동 유지

<br>

## 설치

### 마켓플레이스에서 설치

```bash
/plugin marketplace add gitjay3/self-made-skills
/plugin install decision-logger@self-made-skills
```

<br>

## 사용법

### 자동 제안

Claude가 다음 상황에서 의미 있는 의사결정을 감지하면 자동으로 기록을 제안합니다:

**라이브러리·프레임워크 선택**
- 같은 문제를 해결하는 여러 라이브러리를 비교한 후 하나 채택
- 기존 라이브러리를 다른 것으로 교체 결정
- 새 프레임워크 도입 결정

**데이터·저장소 선택**
- DB 종류 선택 (Postgres/MySQL/MongoDB 등)
- ORM/쿼리 빌더 채택
- 캐싱·검색·메시지큐 솔루션 결정

**아키텍처·패턴 선택**
- monolith vs microservice
- Event Sourcing, CQRS, Hexagonal 등 패턴 적용
- 모듈 경계·계층 구조 변경

**인프라·플랫폼 선택**
- 클라우드 제공자 선택 (AWS/GCP/Azure)
- 컨테이너·오케스트레이션 도구 결정
- CI/CD 파이프라인 도구 변경

**개발 도구·컨벤션**
- 패키지 매니저, 빌드 도구 변경
- 코드 스타일·테스트 전략 표준화

### 수동 시작

자동 제안이 뜨지 않았을 때, 또는 직접 기록하고 싶을 때:

```
스킬 써서 방금 결정한 내용 ADR로 기록해줘
```

```
스킬 써서 우리가 X 선택한 거 정리해줘
```

<br>

## 출력 예시

`docs/adr/0003-use-prisma-orm.md`:

````markdown
# 0003. Prisma를 ORM으로 채택

> 결정일: 2026-04-30
> 상태: Accepted
> 태그: #backend #database #library #high

## Context (배경)

기존에는 raw SQL과 일부 TypeORM이 혼재되어 있어 다음 문제가 있었다:
- 타입 안전성 부족 — 런타임 SQL 에러 빈번
- 마이그레이션 관리가 분산되어 있음
- 신규 입사자 온보딩 시 학습 곡선 가파름

타입 안전한 ORM으로 통일이 필요했음.

## Decision (결정)

**Prisma**를 단일 ORM으로 채택하고, 기존 raw SQL과 TypeORM 코드를 점진적으로 마이그레이션한다.

## Alternatives (검토한 대안)

| 대안 | 장점 | 단점 | 결과 |
|------|------|------|------|
| **Prisma** | 타입 안전성 최고, 마이그레이션 통합, DX 우수 | 복잡한 쿼리 한계, 빌드 단계 추가 | 채택 |
| Drizzle | SQL 친화적, 작은 번들 | 생태계 작음, 마이그레이션 도구 미성숙 | 기각 — 안정성 우선 |
| TypeORM 유지 | 마이그레이션 비용 0 | 타입 안전성 부족, 메인테이너 활동 둔화 | 기각 — 근본 문제 미해결 |
| Kysely | 타입 안전 + SQL 직접 작성 | 마이그레이션 도구 별도 필요 | 기각 — 통합 솔루션 선호 |

## Consequences

### Positive
- 컴파일 시점에 SQL 에러 차단
- 마이그레이션·시드·스키마 한 곳에서 관리
- 자동 생성 타입으로 IDE 지원 강력

### Negative
- 빌드 파이프라인에 `prisma generate` 단계 추가
- 복잡한 윈도우 함수·CTE는 여전히 raw SQL 필요
- 학습 곡선 (스키마 DSL)

### Risks
- Prisma 회사 방향성 변화 시 락인
- 대량 데이터 처리에서 ORM 오버헤드

## 관련 자료

- 공식 문서: https://www.prisma.io/docs
- 벤치마크: 내부 PoC 결과 — `docs/research/orm-benchmark.md`
- 관련 PR: #234, #245
````

<br>

## 태그 가이드

| 카테고리 | 태그 예시 |
|---------|----------|
| 영역 | `#frontend` `#backend` `#database` `#infra` `#api` `#auth` |
| 유형 | `#library` `#framework` `#pattern` `#architecture` `#tooling` `#platform` |
| 영향도 | `#high` `#medium` `#low` |

<br>

## 라이선스

MIT
