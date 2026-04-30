# Learning Logger

새로 배운 개념·API·문법·패턴·도구를 TIL(Today I Learned) 형식으로 자동 기록하는 Claude Code 플러그인입니다.

## 기능

- 새 지식 학습 직후 자동으로 기록 제안
- TL;DR / 무엇을 / 왜 / 어떻게 / 주의할 점 / 관련 자료 구조
- 한 토픽당 한 파일 — 검색·재참조 쉬움
- 태그 기반 분류 (언어/영역/유형/난이도)
- `docs/til/` 폴더에 날짜별 파일로 저장

<br>

## 설치

### 마켓플레이스에서 설치

```bash
/plugin marketplace add gitjay3/self-made-skills
/plugin install learning-logger@self-made-skills
```

<br>

## 사용법

### 자동 제안

Claude가 다음 상황에서 사용자가 **처음 접하는 내용**을 설명한 후 자동으로 기록을 제안합니다:

**언어·문법**
- 처음 보는 언어 문법, 키워드, 연산자 사용법을 설명한 경우
- 새 언어 기능(제네릭, lifetime, decorator 등)을 처음 다룬 경우

**라이브러리·API**
- 처음 사용하는 라이브러리/프레임워크 API를 설명한 경우
- 익숙한 라이브러리의 새 기능·옵션을 알게 된 경우

**디자인 패턴·아키텍처**
- 새 디자인 패턴(Observer, Strategy 등)을 적용한 경우
- 아키텍처 개념(CQRS, Event Sourcing 등)을 처음 다룬 경우

**도구·명령어**
- CLI 명령어, 단축키, IDE 기능을 새로 배운 경우
- 빌드·배포·테스트 도구의 새 옵션을 알게 된 경우

**개념·이론**
- 컴퓨터 과학·웹 표준·프로토콜 등 이론적 개념을 처음 학습한 경우

### 수동 시작

자동 제안이 뜨지 않았을 때, 또는 직접 기록하고 싶을 때 프롬프트에서 요청할 수 있습니다:

```
스킬 써서 방금 배운 내용 TIL로 기록해줘
```

```
스킬 써서 오늘 배운 거 정리해줘
```

<br>

## 출력 예시

`docs/til/2026-04-30-rust-lifetime-elision.md`:

````markdown
# Rust Lifetime Elision 규칙

> 기록일: 2026-04-30
> 태그: #rust #syntax #concept #medium

## TL;DR

Rust 컴파일러는 함수 시그니처에서 lifetime을 3가지 규칙으로 자동 추론하므로, 단순한 케이스는 명시하지 않아도 된다.

## 무엇을 배웠나

Rust에는 함수 시그니처에 lifetime을 매번 명시하지 않아도 되도록 컴파일러가 적용하는 3가지 규칙이 있다:

1. 각 입력 참조 파라미터에 별도의 lifetime이 부여됨
2. 입력 lifetime이 정확히 1개면, 모든 출력 참조에 그 lifetime이 부여됨
3. `&self` 또는 `&mut self`가 있으면, 그 lifetime이 모든 출력에 부여됨

```rust
// 명시적
fn first_word<'a>(s: &'a str) -> &'a str { ... }

// 규칙 2로 elision 가능
fn first_word(s: &str) -> &str { ... }
```

## 왜 중요한가

- 모든 함수에 lifetime을 매번 쓰면 코드가 노이즈로 가득해짐
- 규칙을 알면 **언제 명시가 필요한지**(2개 이상 입력 참조의 출력) 즉시 판단 가능
- 컴파일러 에러 메시지의 "lifetime" 관련 힌트를 빠르게 해석할 수 있음

## 어떻게 적용할까

- 새 함수 작성 시: 입력 참조가 1개면 lifetime 생략
- 컴파일 에러 발생 시: 입력 참조가 2개 이상인지 확인 → 명시 필요
- struct 메서드: `&self`가 있다면 출력에 lifetime 생략 가능

## 주의할 점

- struct 정의 자체에는 elision이 적용되지 않음 — 항상 명시 필요
- 입력 참조 2개 이상에서 출력이 어느 쪽에서 빌려온 것인지 모호하면 컴파일러가 추론 실패

## 관련 자료

- 공식 문서: https://doc.rust-lang.org/reference/lifetime-elision.html
- 관련 파일: `src/parser/tokens.rs`
````

<br>

## 태그 가이드

| 카테고리 | 태그 예시 |
|---------|----------|
| 언어 | `#js` `#ts` `#python` `#go` `#rust` `#java` |
| 영역 | `#frontend` `#backend` `#devops` `#database` `#mobile` |
| 유형 | `#api` `#syntax` `#pattern` `#tool` `#cli` `#shortcut` `#concept` |
| 난이도 | `#easy` `#medium` `#hard` |

<br>

## 라이선스

MIT
