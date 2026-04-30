---
name: log
description: This skill should be used when the user asks to "ADR 기록", "방금 결정한 내용 정리", "기술 결정 문서화", "X 채택한 거 정리", "설계 결정 기록", "log this decision", "document our choice", "record this ADR", or after a significant technical/architectural choice is made (library/framework/database/pattern/architecture/infrastructure/cloud-provider selection). Captures decisions as ADRs in docs/adr/NNNN-title.md following MADR format with Context/Decision/Alternatives/Consequences sections.
argument-hint: "[title (optional)]"
---

# Decision Logger

Capture architectural and technical decisions as ADRs in `docs/adr/`.

## File Creation Rules

1. **Location**: `docs/adr/`
2. **Filename format**: `{NNNN}-{kebab-case-title}.md`
   - NNNN: 4-digit sequence number, starting from 0001
   - Title in English kebab-case
   - Example: `0003-use-prisma-orm.md`
3. **Sequence number**: scan `docs/adr/`, take max number + 1. Start from 0001 if empty.
4. **Create folder if it doesn't exist**
5. **Maintain index**: also update `docs/adr/README.md` with a list of all ADRs (number, title, status, date)

## Workflow (4 Steps)

### Step 1: 정보 수집

Gather these 5 pieces from the conversation context. Ask the user for any that are missing — never invent:

1. **결정 내용** — 정확히 무엇을 선택했나
2. **배경** — 왜 결정이 필요했나, 어떤 문제/제약이 있었나
3. **검토한 대안** — 고려한 다른 옵션들과 기각 사유
4. **트레이드오프** — 선택의 장단점, 리스크
5. **참고 자료** — 문서, 벤치마크, PR/이슈 링크 등

### Step 2: 시퀀스 번호 결정

Glob `docs/adr/*.md` → 파일명에서 NNNN 추출 → max + 1. 빈 디렉토리면 `0001`.

### Step 3: ADR 작성

Generate the ADR document in Korean using this template:

```markdown
# {NNNN}. [결정 제목]

> 결정일: YYYY-MM-DD
> 상태: Proposed | Accepted | Deprecated | Superseded by ADR-XXXX
> 태그: #카테고리1 #카테고리2

## Context (배경)

이 결정이 왜 필요했나? 어떤 상황·문제·제약이 있었나?
- 기술적 제약
- 비즈니스 요구사항
- 팀 상황

## Decision (결정)

무엇을 선택했나? 명확하게 한두 문장.

## Alternatives (검토한 대안)

| 대안 | 장점 | 단점 | 결과 |
|------|------|------|------|
| **선택안** | ... | ... | 채택 |
| 대안 A | ... | ... | 기각 — 사유 |
| 대안 B | ... | ... | 기각 — 사유 |

## Consequences (결과·영향)

### Positive
- 어떤 이점이 있는가
- 어떤 문제가 해결되는가

### Negative
- 어떤 비용·제약이 생기는가
- 어떤 추가 작업이 필요한가

### Risks
- 잘못될 경우 어떤 리스크가 있는가
- 이 결정이 깨지는 조건은 무엇인가

## 결정하지 않은 것 (선택)

명시적으로 미루거나 범위에서 제외한 것들. 없으면 섹션 자체를 생략.

## 관련 자료

- 공식 문서:
- 벤치마크·리서치:
- 관련 PR·이슈:
- 관련 파일: `src/example.ts`
```

### Step 4: 리뷰 제시

작성된 드래프트를 사용자에게 보여주고:
- 빠진 정보가 있는지 확인
- 사실 관계가 정확한지 확인
- 사용자 컨펌 후 파일 저장 + index README 업데이트

## Status Lifecycle

- **Proposed**: 제안 단계, 합의 전
- **Accepted**: 채택되어 적용 중
- **Deprecated**: 더 이상 권장하지 않음 (대체재 없음)
- **Superseded by ADR-XXXX**: 새 ADR로 대체됨

기존 ADR을 대체하는 결정이라면, 새 ADR을 작성하면서 기존 ADR의 상태를 `Superseded by ADR-NNNN`으로 업데이트할지 사용자에게 묻는다.

## Tag Guide

| Category | Examples |
|----------|----------|
| Area | `#frontend` `#backend` `#database` `#infra` `#api` `#auth` |
| Type | `#library` `#framework` `#pattern` `#architecture` `#tooling` `#platform` |
| Impact | `#high` `#medium` `#low` |

## Writing Guidelines

1. **Capture rationale, not just the decision** — 6개월 후 읽어도 "왜 그랬는지" 즉시 이해 가능해야 함
2. **Document rejected alternatives honestly** — 왜 떨어졌는지가 핵심 가치
3. **Be brief** — 1~3페이지 분량. 더 길어지면 RFC 문서로 옮길 신호
4. **Avoid vague language** — "더 낫다", "좋다" 대신 구체적 근거(벤치마크, 비용, 학습 곡선 등)
5. **Update status when superseded** — 결정이 바뀌면 새 ADR + 기존 ADR 상태 갱신

## Invocation

### Automatic
When a significant technical/architectural decision is made (library/framework/DB/pattern selection), suggest "방금 결정한 내용을 ADR로 기록해두시겠어요?"

### Manual
```
/decision-logger:log
/decision-logger:log use-prisma-orm
```
