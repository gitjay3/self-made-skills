---
name: log
description: This skill should be used when the user asks to "TIL 기록", "방금 배운 내용 정리", "오늘 배운 거 정리", "TIL로 기록", "log this learning", "save what I learned", or after Claude explains a new concept/API/syntax/library/pattern/tool/command/shortcut to the user for the first time. Captures learnings as TIL (Today I Learned) entries in docs/til/YYYY-MM-DD-topic.md with TL;DR/What/Why/How/Pitfalls structure.
argument-hint: "[title (optional)]"
---

# Learning Logger

Capture what you just learned as a TIL entry in `docs/til/`.

## File Creation Rules

1. **Location**: `docs/til/`
2. **Filename format**: `YYYY-MM-DD-topic.md`
   - Topic should be in English kebab-case
   - Example: `2026-04-30-rust-lifetime-elision.md`
3. **Create folder if it doesn't exist**

## Output Template (Korean)

Generate the TIL document in Korean using this template:

```markdown
# [배운 주제]

> 기록일: YYYY-MM-DD
> 태그: #카테고리1 #카테고리2

## TL;DR

6개월 후 본인이 다시 봐도 핵심이 떠오르도록 1~2문장으로 요약.

## 무엇을 배웠나

구체적인 내용을 정리합니다.
- 개념/API/문법/패턴/명령어가 무엇인가?
- 어떻게 동작하는가?

```예시 코드 (있다면)```

## 왜 중요한가

- 이 지식이 어떤 문제를 해결하는가?
- 모르고 있었다면 어떤 비용이 있었을까?
- 기존에 알던 지식과 어떻게 연결되는가?

## 어떻게 적용할까

실제로 언제 어떻게 쓸지 시나리오 1~2개.

## 주의할 점 (선택)

함정, 한계, 예외 케이스 — 있을 때만 작성.

## 관련 자료

- 공식 문서:
- 발견한 곳:
- 관련 파일: `src/example.ts`
```

## Tag Guide

Select appropriate tags for the topic:

| Category | Examples |
|----------|----------|
| Language | `#js` `#ts` `#python` `#go` `#rust` `#java` |
| Area | `#frontend` `#backend` `#devops` `#database` `#mobile` |
| Type | `#api` `#syntax` `#pattern` `#tool` `#cli` `#shortcut` `#concept` |
| Difficulty | `#easy` `#medium` `#hard` |

## Writing Guidelines

1. **Auto-generate from conversation**: Base the document on what was just explained or learned in this session
2. **One topic per file**: Keep TIL entries focused on a single learning. Split if multiple unrelated learnings came up
3. **Be specific**: Include actual code, exact commands, and concrete examples — not abstract paraphrases
4. **Future-self test**: Write so that you (6 months later) can grasp the point in 30 seconds
5. **Skip optional sections**: If "주의할 점" doesn't apply, omit it entirely instead of leaving it empty

## Invocation

### Automatic
After Claude explains a new concept/API/pattern/tool/command, suggest "방금 배운 내용을 TIL로 기록해두시겠어요?"

### Manual
```
/learning-logger:log
/learning-logger:log rust-lifetime-elision
```
