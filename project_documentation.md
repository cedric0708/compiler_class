# 간단한 프로그래밍 언어 컴파일러/인터프리터 프로젝트 설명서

## 1. 언어 개요 및 설계 의도

### 1.1 언어 개요

이 프로젝트는 교육용으로 설계된 간단한 프로그래밍 언어입니다. 이 언어는 컴파일러 이론의 핵심 개념을 학습하기 위해 개발되었으며, 최소한의 문법으로도 실용적인 프로그램을 작성할 수 있도록 설계되었습니다.

### 1.2 설계 의도

1. **교육 목적**: 컴파일러의 기본 구성 요소(어휘 분석, 구문 분석, 실행)를 명확하게 이해할 수 있도록 구현
2. **간결성**: 복잡한 기능보다는 핵심 기능에 집중하여 언어의 구조를 쉽게 파악할 수 있도록 설계
3. **실용성**: 변수, 조건문, 반복문, 함수 등 프로그래밍의 기본 요소를 모두 포함하여 실제 프로그램 작성 가능
4. **확장성**: 향후 배열, 객체, 타입 시스템 등을 추가할 수 있는 구조로 설계

### 1.3 언어 특징

- **동적 타입**: 변수의 타입을 명시적으로 선언할 필요 없음
- **C/JavaScript 스타일 문법**: 널리 사용되는 언어와 유사한 문법으로 학습 곡선 최소화
- **인터프리터 방식**: AST를 직접 실행하여 즉시 결과 확인 가능

## 2. 문법 정의 (Grammar)

### 2.1 EBNF 문법 정의

```
Program     ::= Statement*

Statement   ::= VariableDeclaration
             |  Assignment
             |  IfStatement
             |  WhileStatement
             |  FunctionDefinition
             |  ReturnStatement
             |  PrintStatement
             |  ExpressionStatement

VariableDeclaration ::= "let" IDENTIFIER ["=" Expression] ";"

Assignment  ::= IDENTIFIER "=" Expression ";"

IfStatement ::= "if" "(" Expression ")" "{" Statement* "}" ["else" "{" Statement* "}"]

WhileStatement ::= "while" "(" Expression ")" "{" Statement* "}"

FunctionDefinition ::= "function" IDENTIFIER "(" [IDENTIFIER ("," IDENTIFIER)*] ")" "{" Statement* "}"

ReturnStatement ::= "return" [Expression] ";"

PrintStatement ::= "print" Expression ";"

ExpressionStatement ::= Expression ";"

Expression  ::= OrExpression

OrExpression ::= AndExpression (("||") AndExpression)*

AndExpression ::= EqualityExpression (("&&") EqualityExpression)*

EqualityExpression ::= ComparisonExpression (("==" | "!=") ComparisonExpression)*

ComparisonExpression ::= Term (("<" | "<=" | ">" | ">=") Term)*

Term        ::= Factor (("+" | "-") Factor)*

Factor      ::= Unary (("*" | "/" | "%") Unary)*

Unary       ::= ("!" | "-") Unary | Primary

Primary     ::= NUMBER
             |  STRING
             |  IDENTIFIER ["(" [Expression ("," Expression)*] ")"]
             |  "(" Expression ")"

NUMBER      ::= DIGIT+ ["." DIGIT+]

STRING      ::= '"' (CHAR | ESCAPE_SEQUENCE)* '"'

IDENTIFIER  ::= (LETTER | "_") (LETTER | DIGIT | "_")*

LETTER      ::= "a" | "b" | ... | "z" | "A" | "B" | ... | "Z"

DIGIT       ::= "0" | "1" | ... | "9"

ESCAPE_SEQUENCE ::= "\\" ("n" | "t" | "\\" | '"')

COMMENT     ::= "//" (CHAR - NEWLINE)* NEWLINE
```

### 2.2 토큰 정의

#### 키워드
- `let`: 변수 선언
- `if`: 조건문
- `else`: 조건문의 else 절
- `while`: 반복문
- `function`: 함수 정의
- `return`: 함수 반환
- `print`: 출력문

#### 연산자
- 산술: `+`, `-`, `*`, `/`, `%`
- 비교: `==`, `!=`, `<`, `<=`, `>`, `>=`
- 논리: `&&`, `||`, `!`
- 대입: `=`

#### 구분자
- `;`: 문장 종료
- `,`: 매개변수/인자 구분
- `(`, `)`: 표현식/함수 호출
- `{`, `}`: 블록

#### 리터럴
- 숫자: 정수 및 실수 (예: `42`, `3.14`)
- 문자열: 큰따옴표로 감싼 문자 시퀀스 (예: `"hello"`)

### 2.3 연산자 우선순위

우선순위가 높은 것부터 낮은 것 순서:

1. 단항 연산자: `!`, `-`
2. 곱셈/나눗셈/나머지: `*`, `/`, `%`
3. 덧셈/뺄셈: `+`, `-`
4. 비교: `<`, `<=`, `>`, `>=`
5. 동등 비교: `==`, `!=`
6. 논리 AND: `&&`
7. 논리 OR: `||`

## 3. 전체 구조

### 3.1 컴파일러 파이프라인

```
소스 코드
    ↓
[Lexer] 어휘 분석
    ↓
토큰 스트림
    ↓
[Parser] 구문 분석
    ↓
AST (Abstract Syntax Tree)
    ↓
[Interpreter] 실행
    ↓
프로그램 결과
```

### 3.2 각 단계 상세 설명

#### 3.2.1 어휘 분석 (Lexer)

**역할**: 소스 코드를 토큰으로 분해

**입력**: 문자열 형태의 소스 코드
**출력**: 토큰 리스트

**주요 기능**:
- 키워드 인식 (`let`, `if`, `while`, 등)
- 식별자 인식 (변수명, 함수명)
- 숫자 리터럴 파싱 (정수, 실수)
- 문자열 리터럴 파싱 (이스케이프 시퀀스 처리)
- 연산자 및 구분자 인식
- 공백 및 주석 제거 (`//` 형태 주석 지원)

**구현 파일**: `lexer.py`

#### 3.2.2 구문 분석 (Parser)

**역할**: 토큰 스트림을 AST로 변환

**입력**: 토큰 리스트
**출력**: AST (Program 노드)

**파싱 방식**: 재귀 하강 파서 (Recursive Descent Parser)

**주요 기능**:
- 표현식 파싱 (연산자 우선순위 고려)
- 문장 파싱 (변수 선언, 조건문, 반복문, 함수 등)
- 블록 파싱
- 구문 오류 감지 및 에러 메시지 생성

**구현 파일**: `parser.py`

#### 3.2.3 AST (Abstract Syntax Tree)

**역할**: 프로그램의 구조를 트리 형태로 표현

**주요 노드 타입**:

**표현식 노드**:
- `NumberLiteral`: 숫자 리터럴
- `StringLiteral`: 문자열 리터럴
- `Identifier`: 변수 참조
- `BinaryOp`: 이항 연산
- `UnaryOp`: 단항 연산
- `FunctionCall`: 함수 호출

**문장 노드**:
- `VariableDeclaration`: 변수 선언
- `Assignment`: 변수 대입
- `IfStatement`: 조건문
- `WhileStatement`: 반복문
- `FunctionDefinition`: 함수 정의
- `ReturnStatement`: 반환문
- `PrintStatement`: 출력문
- `ExpressionStatement`: 표현식 문장

**프로그램 노드**:
- `Program`: 프로그램 루트 노드

**구현 파일**: `ast.py`

#### 3.2.4 인터프리터 (Interpreter)

**역할**: AST를 순회하며 프로그램을 실행

**입력**: AST (Program 노드)
**출력**: 프로그램 실행 결과

**주요 기능**:
- 변수 환경 관리 (Environment)
  - 스코프 체인을 통한 변수 검색
  - 블록별 독립적인 환경 생성
- 표현식 평가
  - 산술/비교/논리 연산 실행
  - 함수 호출 처리
- 문장 실행
  - 변수 선언/대입
  - 조건문/반복문 제어 흐름
  - 함수 정의 및 호출
- 에러 처리
  - 런타임 오류 감지 및 보고

**구현 파일**: `interpreter.py`

### 3.3 실행 흐름도

```
[main.py]
    ↓
run(source)
    ↓
┌─────────────────┐
│  Lexer.tokenize()│ → 토큰 리스트
└─────────────────┘
    ↓
┌─────────────────┐
│  Parser.parse() │ → AST (Program)
└─────────────────┘
    ↓
┌──────────────────────┐
│ Interpreter.interpret()│
│  ┌──────────────────┐ │
│  │ _execute()      │ │ → 각 문장 실행
│  │  └─ _evaluate()  │ │ → 표현식 평가
│  └──────────────────┘ │
└──────────────────────┘
    ↓
프로그램 결과 출력
```

### 3.4 변수 환경 (Environment)

인터프리터는 변수를 관리하기 위해 **Environment** 클래스를 사용합니다.

**특징**:
- 스코프 체인: 부모 환경을 참조하여 상위 스코프의 변수 접근
- 블록별 환경: 각 블록(`{}`)마다 새로운 환경 생성
- 클로저 지원: 함수는 정의된 시점의 환경을 기억

**구조**:
```
Global Environment
    ↓
Block Environment 1
    ↓
Block Environment 2
    ...
```

## 4. 구현된 기능

### 4.1 필수 기능 (과제 요구사항)

✅ **변수 선언 및 대입**
- `let` 키워드를 사용한 변수 선언
- 초기값 지정 가능
- 변수 재할당 지원

✅ **산술 연산**
- `+`, `-`, `*`, `/`, `%` 연산자 지원
- 정수 및 실수 연산

✅ **비교 연산**
- `==`, `!=`, `<`, `<=`, `>`, `>=` 연산자 지원

✅ **논리 연산**
- `&&`, `||`, `!` 연산자 지원
- 단락 평가 (short-circuit evaluation) 지원

✅ **조건문**
- `if` / `if-else` 문 지원
- 중첩 조건문 지원

✅ **반복문**
- `while` 문 지원
- 중첩 반복문 지원

✅ **함수 호출**
- 함수 정의 및 호출 지원
- 매개변수 전달 지원
- 반환값 지원 (`return` 문)
- 클로저 지원 (외부 변수 접근)

### 4.2 추가 구현 기능

✅ **출력문**
- `print` 문을 통한 값 출력

✅ **문자열 리터럴**
- 문자열 타입 지원
- 이스케이프 시퀀스 지원 (`\n`, `\t`, `\\`, `\"`)

✅ **주석**
- `//` 형태의 한 줄 주석 지원

✅ **에러 처리**
- 구문 오류 감지 및 위치 정보 제공
- 런타임 오류 감지 및 메시지 제공

✅ **REPL 모드**
- 대화형 실행 환경 제공

## 5. 미구현/제한 사항

### 5.1 미구현 기능

❌ **배열/리스트**
- 배열 타입 미지원
- 인덱싱 연산 미지원

❌ **객체/클래스**
- 객체 지향 프로그래밍 미지원
- 구조체/레코드 타입 미지원

❌ **타입 시스템**
- 정적 타입 검사 미지원
- 타입 선언 미지원

❌ **고급 제어 구조**
- `for` 반복문 미지원
- `switch` 문 미지원
- `break` / `continue` 문 미지원

❌ **문자열 연산**
- 문자열 연결 연산자 미지원 (숫자만 `+` 연산 가능)
- 문자열 슬라이싱 미지원

❌ **내장 함수**
- `print` 외의 내장 함수 미지원
- 수학 함수 (`sin`, `cos`, 등) 미지원
- 입출력 함수 미지원

❌ **모듈 시스템**
- 파일 간 모듈 import 미지원
- 네임스페이스 미지원

### 5.2 제한 사항

⚠️ **재귀 함수 호출**
- 재귀 호출이 가능하지만 깊은 재귀 시 스택 오버플로우 발생 가능
- 최대 재귀 깊이 제한 없음

⚠️ **타입 안전성**
- 동적 타입으로 인한 런타임 타입 오류 가능
- 타입 변환 자동 처리 (숫자와 문자열 혼용 시 오류)

⚠️ **메모리 관리**
- 가비지 컬렉션 없음 (Python의 GC에 의존)
- 메모리 누수 가능성 (순환 참조 시)

⚠️ **성능**
- 인터프리터 방식으로 인한 실행 속도 제한
- 최적화 없음 (AST를 직접 실행)

⚠️ **에러 복구**
- 구문 오류 발생 시 파싱 중단 (부분 파싱 불가)
- 에러 복구 메커니즘 없음

### 5.3 알려진 버그/이슈

- 없음 (현재까지 발견된 버그 없음)

## 6. 테스트 프로그램

테스트 프로그램은 `examples/` 디렉토리에 위치하며, 총 **20개의 테스트 프로그램**이 포함되어 있습니다. 각 테스트의 목적과 기대 결과는 `examples/TEST_DOCUMENTATION.md`에 상세히 정리되어 있습니다.

### 테스트 목록

1. **test01_basic_variables.lang**: 변수 선언 및 대입
2. **test02_arithmetic.lang**: 산술 연산
3. **test03_comparison.lang**: 비교 연산
4. **test04_logical.lang**: 논리 연산
5. **test05_if_statement.lang**: 조건문 (if)
6. **test06_if_else.lang**: 조건문 (if-else)
7. **test07_while_loop.lang**: 반복문 (while)
8. **test08_factorial.lang**: 팩토리얼 계산
9. **test09_function.lang**: 함수 정의 및 호출
10. **test10_function_multiple.lang**: 여러 함수
11. **test11_closure.lang**: 클로저
12. **test12_nested_if.lang**: 중첩 조건문
13. **test13_nested_while.lang**: 중첩 반복문
14. **test14_complex_expression.lang**: 복잡한 표현식
15. **test15_string.lang**: 문자열 리터럴
16. **test16_unary_operators.lang**: 단항 연산자
17. **test17_fibonacci.lang**: 피보나치 수열
18. **test18_function_recursion.lang**: 재귀 함수
19. **test19_operator_precedence.lang**: 연산자 우선순위
20. **test20_complex_program.lang**: 복합 프로그램

### 테스트 카테고리

1. **기본 기능 테스트** (6개)
   - test01_basic_variables.lang: 변수 선언 및 대입
   - test02_arithmetic.lang: 산술 연산
   - test03_comparison.lang: 비교 연산
   - test04_logical.lang: 논리 연산
   - test15_string.lang: 문자열 리터럴
   - test16_unary_operators.lang: 단항 연산자

2. **제어 흐름 테스트** (5개)
   - test05_if_statement.lang: 조건문 (if)
   - test06_if_else.lang: 조건문 (if-else)
   - test07_while_loop.lang: 반복문 (while)
   - test12_nested_if.lang: 중첩 조건문
   - test13_nested_while.lang: 중첩 반복문

3. **함수 테스트** (4개)
   - test09_function.lang: 함수 정의 및 호출
   - test10_function_multiple.lang: 여러 함수
   - test11_closure.lang: 클로저
   - test18_function_recursion.lang: 재귀 함수

4. **복합 기능 테스트** (5개)
   - test08_factorial.lang: 팩토리얼 계산
   - test14_complex_expression.lang: 복잡한 표현식
   - test17_fibonacci.lang: 피보나치 수열
   - test19_operator_precedence.lang: 연산자 우선순위
   - test20_complex_program.lang: 복합 프로그램

## 7. 향후 개선 방향

1. **타입 시스템 추가**
   - 정적 타입 검사
   - 타입 추론

2. **배열/리스트 지원**
   - 배열 리터럴
   - 인덱싱 연산
   - 배열 내장 함수

3. **객체 지향 프로그래밍**
   - 클래스 정의
   - 객체 생성 및 메서드 호출

4. **성능 최적화**
   - 바이트코드 생성
   - JIT 컴파일

5. **표준 라이브러리**
   - 수학 함수
   - 문자열 처리 함수
   - 입출력 함수

## 8. 참고 자료

- 컴파일러 이론 교재
- Python 공식 문서
- EBNF 문법 표기법

---

**작성일**: 2024년
**버전**: 1.0



## 8. 결론

이 프로젝트는 컴파일러의 기본 구조인 어휘 분석, 구문 분석, 실행 단계를 모두 구현한 완전한 인터프리터입니다. 교육 목적에 맞게 코드를 명확하게 작성하고, 각 단계를 모듈화하여 이해하기 쉽게 구성했습니다.

이 언어는 간단하지만 필수적인 프로그래밍 개념을 모두 포함하고 있어, 컴파일러 이론 학습에 적합한 프로젝트입니다.

