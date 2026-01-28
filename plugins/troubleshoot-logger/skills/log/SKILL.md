---
name: log
description: Automatically logs troubleshooting process using STAR method when errors or bugs are resolved. Use this skill to document problem-solving sessions.
argument-hint: "[title (optional)]"
---

# Troubleshoot Logger

Log troubleshooting sessions using STAR method + Root Cause Analysis to `docs/troubleshooting/`.

## File Creation Rules

1. **Location**: `docs/troubleshooting/`
2. **Filename format**: `YYYY-MM-DD-title.md`
   - Title should be in English kebab-case
   - Example: `2026-01-28-api-latency-fix.md`
3. **Create folder if it doesn't exist**

## Output Template (Korean)

Generate the troubleshooting document in Korean using this template:

```markdown
# [문제 제목]

> 기록일: YYYY-MM-DD
> 태그: #카테고리1 #카테고리2

## Situation (상황)

문제가 발생한 상황과 배경을 설명합니다.
- 어떤 환경에서 발생했는가?
- 어떤 증상이 있었는가?
- 에러 메시지나 로그 포함

## Task (과제)

해결해야 할 목표를 명확히 합니다.
- 무엇을 달성해야 하는가?
- 제약 조건이 있는가?

## Action (행동)

문제 해결을 위해 취한 조치들을 단계별로 기록합니다.
- 어떤 가설을 세웠는가?
- 어떤 시도를 했는가?
- 최종적으로 어떤 해결책을 적용했는가?

## Result (결과)

결과를 정리합니다.
- 문제가 해결되었는가?
- 성능/품질 개선 수치 (있다면)

## Root Cause (근본 원인 분석)

5 Whys 기법으로 근본 원인을 파악합니다.

1. **Why**: 왜 문제가 발생했는가? → [직접적 원인]
2. **Why**: 왜 그런 상황이 생겼는가? → [중간 원인]
3. **Why**: 왜 그것이 가능했는가? → [근본 원인]

## Prevention (재발 방지)

향후 같은 문제를 예방하기 위한 조치:
- [ ] 조치 1
- [ ] 조치 2

## Related (관련 자료)

- 파일: `src/example/file.ts`
- 커밋: `abc1234`
- PR/이슈: #123
```

## Tag Guide

Select appropriate tags for the issue:

| Category | Examples |
|----------|----------|
| Area | `#backend` `#frontend` `#database` `#infra` `#auth` `#api` |
| Type | `#bug` `#performance` `#security` `#config` `#dependency` |
| Difficulty | `#easy` `#medium` `#hard` |

## Writing Guidelines

1. **Auto-generate from conversation**: Base the document on the current session's problem-solving process
2. **Be specific**: Include actual code, commands, and error messages rather than abstract descriptions
3. **Make it reproducible**: Write so that someone facing the same issue can understand
4. **Use code blocks**: Format changed code and executed commands as code blocks
5. **5 Whys depth**: Usually 3 levels is enough to reach root cause

## Invocation

### Automatic
When Claude completes fixing an error/bug, automatically suggest "트러블슈팅 내용을 기록할까요?"

### Manual
```
/troubleshoot-logger:log
/troubleshoot-logger:log api-timeout-fix
```
