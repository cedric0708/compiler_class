# 컴파일러/인터프리터 프로젝트 설명서

## 1. 프로젝트 개요

### 1.1 언어 소개

이 프로젝트는 교육용으로 설계된 간단한 프로그래밍 언어의 컴파일러/인터프리터를 구현합니다. 이 언어는 컴파일러 이론의 핵심 개념인 어휘 분석(Lexical Analysis), 구문 분석(Syntax Analysis), 그리고 코드 실행(Interpretation)을 학습하기 위한 목적으로 개발되었습니다.

### 1.2 설계 의도

- **간결성**: 최소한의 문법으로 핵심 프로그래밍 개념을 표현
- **교육성**: 컴파일러 구조를 이해하기 쉽게 구현
- **확장성**: 향후 기능 추가가 용이한 구조

### 1.3 구현된 기능

- ✅ 변수 선언 및 대입
- ✅ 산술 연산 (+, -, *, /, %)
- ✅ 비교 연산 (==, !=, <, <=, >, >=)
- ✅ 논리 연산 (&&, ||, !)
- ✅ 조건문 (if / if-else)
- ✅ 반복문 (while)
- ✅ 함수 정의 및 호출
- ✅ 문자열 리터럴
- ✅ 주석 처리

## 2. 언어 문법 (Grammar)

### 2.1 EBNF 문법 정의

```
Program     = { Statement } EOF

Statement   = VariableDeclaration
            | Assignment
            | IfStatement
            | WhileStatement
            | FunctionDefinition
            | ReturnStatement
            | PrintStatement
            | ExpressionStatement

VariableDeclaration = "let" IDENTIFIER [ "=" Expression ] ";"

Assignment  = IDENTIFIER "=" Expression ";"

IfStatement = "if" "(" Expression ")" "{" Statement* "}" [ "else" "{" Statement* "}" ]

WhileStatement = "while" "(" Expression ")" "{" Statement* "}"

FunctionDefinition = "function" IDENTIFIER "(" [ IDENTIFIER { "," IDENTIFIER } ] ")" "{" Statement* "}"

ReturnStatement = "return" [ Expression ] ";"

PrintStatement = "print" Expression ";"

ExpressionStatement = Expression ";"

Expression  = OrExpression

OrExpression = AndExpression { "||" AndExpression }

AndExpression = EqualityExpression { "&&" EqualityExpression }

EqualityExpression = ComparisonExpression { ("==" | "!=") ComparisonExpression }

ComparisonExpression = Term { ("<" | "<=" | ">" | ">=") Term }

Term        = Factor { ("+" | "-") Factor }

Factor      = Unary { ("*" | "/" | "%") Unary }

Unary       = ("!" | "-") Unary | Primary

Primary     = NUMBER
            | STRING
            | IDENTIFIER [ "(" [ Expression { "," Expression } ] ")" ]
            | "(" Expression ")"

NUMBER      = DIGIT+ [ "." DIGIT+ ]

STRING      = '"' { CHAR | ESCAPE } '"'

IDENTIFIER  = LETTER { LETTER | DIGIT | "_" }

LETTER      = "a" | "b" | ... | "z" | "A" | "B" | ... | "Z"

DIGIT       = "0" | "1" | ... | "9"

CHAR        = 모든 문자 (", \ 제외)

ESCAPE      = "\\n" | "\\t" | "\\\\" | "\\\""

COMMENT     = "//" { CHAR } NEWLINE
```

### 2.2 토큰 정의

#### 키워드
- `let`: 변수 선언
- `if`: 조건문 시작
- `else`: 조건문의 else 분기
- `while`: 반복문
- `function`: 함수 정의
- `return`: 함수 반환
- `print`: 출력

#### 연산자
- 산술: `+`, `-`, `*`, `/`, `%`
- 비교: `==`, `!=`, `<`, `<=`, `>`, `>=`
- 논리: `&&`, `||`, `!`
- 대입: `=`

#### 구분자
- `;`: 문장 종료
- `,`: 인자/매개변수 구분
- `(`, `)`: 표현식/함수 호출
- `{`, `}`: 블록

#### 리터럴
- 숫자: 정수 또는 부동소수점
- 문자열: `"..."` 형태

#### 식별자
- 문자, 숫자, 언더스코어(`_`)로 구성
- 숫자로 시작할 수 없음

### 2.3 연산자 우선순위

우선순위가 높을수록 먼저 평가됩니다:

1. 단항 연산 (`!`, `-`)
2. 곱셈/나눗셈/나머지 (`*`, `/`, `%`)
3. 덧셈/뺄셈 (`+`, `-`)
4. 비교 (`<`, `<=`, `>`, `>=`)
5. 동등 비교 (`==`, `!=`)
6. 논리 AND (`&&`)
7. 논리 OR (`||`)

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
실행 결과
```

### 3.2 모듈 구조

#### 3.2.1 Lexer (lexer.py)

**역할**: 소스 코드를 토큰 스트림으로 변환

**주요 클래스/함수**:
- `TokenType`: 토큰 타입 열거형
- `Token`: 토큰 클래스 (타입, 값, 위치 정보)
- `Lexer`: 어휘 분석기
  - `tokenize()`: 토큰화 메인 함수
  - `read_number()`: 숫자 토큰 읽기
  - `read_string()`: 문자열 토큰 읽기
  - `read_identifier()`: 식별자/키워드 읽기
  - `skip_comment()`: 주석 건너뛰기

**처리 과정**:
1. 소스 코드를 문자 단위로 읽음
2. 공백/주석 건너뛰기
3. 문자 패턴에 따라 토큰 생성
4. 토큰 리스트 반환

#### 3.2.2 Parser (parser.py)

**역할**: 토큰 스트림을 AST로 변환

**주요 클래스/함수**:
- `Parser`: 구문 분석기
  - `parse()`: 파싱 메인 함수
  - `expression()`: 표현식 파싱
  - `statement()`: 문장 파싱
  - `function_definition()`: 함수 정의 파싱
  - 재귀 하강 파서 (Recursive Descent Parser) 방식 사용

**파싱 전략**:
- 재귀 하강 파싱 (Recursive Descent Parsing)
- 연산자 우선순위에 따른 표현식 파싱
- 오류 발생 시 `ParseError` 예외 발생

#### 3.2.3 AST (ast.py)

**역할**: 추상 구문 트리 노드 정의

**노드 타입**:

**표현식 노드**:
- `NumberLiteral`: 숫자 리터럴
- `StringLiteral`: 문자열 리터럴
- `Identifier`: 변수명
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
- `Program`: 프로그램 루트

#### 3.2.4 Interpreter (interpreter.py)

**역할**: AST를 실행하여 프로그램 평가

**주요 클래스/함수**:
- `Environment`: 변수 환경 (스코프 관리)
- `Function`: 함수 객체
- `Interpreter`: 인터프리터
  - `interpret()`: 프로그램 실행
  - `_execute()`: 문장 실행
  - `_evaluate()`: 표현식 평가
  - `_execute_block()`: 블록 실행 (스코프 생성)

**실행 과정**:
1. AST를 순회하며 노드 타입에 따라 처리
2. 표현식 평가: 리터럴 값 반환, 연산 수행
3. 문장 실행: 변수 정의, 제어 흐름 처리
4. 환경 관리: 변수 스코프 유지

**스코프 관리**:
- 각 블록마다 새로운 `Environment` 생성
- 부모 환경에 대한 참조 유지 (클로저 지원)
- 변수 검색 시 현재 환경부터 부모 환경 순으로 탐색

### 3.3 실행 흐름도

```
[사용자 입력]
    ↓
[main.py] 파일 읽기 또는 REPL 입력
    ↓
[Lexer] 소스 코드 → 토큰 스트림
    ↓
[Parser] 토큰 스트림 → AST
    ↓
[Interpreter] AST 실행
    ├─ [Environment] 변수 환경 관리
    ├─ [Expression Evaluation] 표현식 평가
    └─ [Statement Execution] 문장 실행
    ↓
[출력] 결과 표시
```

## 4. 구현 세부사항

### 4.1 어휘 분석 (Lexical Analysis)

#### 토큰화 알고리즘

1. **상태 기반 스캔**: 현재 문자에 따라 다음 동작 결정
2. **문자 클래스별 처리**:
   - 숫자: `read_number()` - 정수/부동소수점 구분
   - 문자: `read_identifier()` - 키워드/식별자 구분
   - 따옴표: `read_string()` - 이스케이프 시퀀스 처리
3. **주석 처리**: `//` 발견 시 줄 끝까지 건너뛰기
4. **위치 정보 유지**: 각 토큰에 줄 번호, 열 번호 저장

#### 특수 처리

- **이스케이프 시퀀스**: `\n`, `\t`, `\\`, `\"` 지원
- **다중 문자 연산자**: `==`, `!=`, `<=`, `>=`, `&&`, `||` 처리
- **공백 건너뛰기**: 의미 없는 공백/탭 무시

### 4.2 구문 분석 (Syntax Analysis)

#### 파싱 전략

**재귀 하강 파서 (Recursive Descent Parser)**:
- 각 문법 규칙에 대응하는 함수 구현
- 좌측 결합(left-associative) 연산자 처리
- 연산자 우선순위에 따른 함수 호출 순서

#### 표현식 파싱

**Pratt 파서 스타일**:
- 우선순위가 낮은 연산자부터 높은 연산자 순으로 파싱
- `or_expression()` → `and_expression()` → `equality_expression()` → ... → `primary()`

#### 오류 처리

- **예상 토큰 불일치**: `consume()` 함수에서 `ParseError` 발생
- **위치 정보 포함**: 오류 발생 위치(줄, 열) 표시
- **부분 복구**: 가능한 경우 파싱 계속 진행

### 4.3 실행 (Interpretation)

#### 평가 전략

**트리 순회 (Tree Walking)**:
- AST를 재귀적으로 순회하며 평가
- 각 노드 타입에 따라 적절한 처리 수행

#### 표현식 평가

1. **리터럴**: 값 그대로 반환
2. **식별자**: 환경에서 변수 값 조회
3. **이항 연산**: 좌우 피연산자 평가 후 연산 수행
4. **단항 연산**: 피연산자 평가 후 연산 수행
5. **함수 호출**: 인자 평가 → 함수 실행 → 반환값

#### 문장 실행

1. **변수 선언**: 환경에 변수 정의
2. **변수 대입**: 환경에서 변수 찾아 값 할당
3. **조건문**: 조건 평가 → 분기 선택 → 블록 실행
4. **반복문**: 조건 평가 → 참이면 블록 실행 → 반복
5. **함수 정의**: 함수 객체 생성 → 환경에 저장
6. **반환문**: `ReturnValue` 예외 발생 → 함수 호출자에게 전달

#### 환경 관리

**스코프 체인**:
```
Global Environment
    ↓
Function Environment (클로저)
    ↓
Block Environment
```

- 각 블록/함수마다 새로운 환경 생성
- 부모 환경에 대한 참조 유지
- 변수 검색: 현재 → 부모 → ... → 전역 순서

### 4.4 타입 시스템

**동적 타입 (Dynamic Typing)**:
- 변수 타입 선언 불필요
- 실행 시점에 타입 결정
- 지원 타입: 정수, 부동소수점, 문자열, 불리언(암묵적), 함수, None

**타입 변환**:
- 산술 연산: 숫자 타입만 허용
- 비교 연산: 숫자 비교 또는 동등성 비교
- 논리 연산: 진리값 변환 후 평가

## 5. 테스트 프로그램

### 5.1 테스트 목록

총 15개의 테스트 프로그램이 포함되어 있으며, 각각 다른 기능을 검증합니다:

1. **test01_basic_arithmetic.amoro**: 기본 산술 연산
2. **test02_variables.amoro**: 변수 선언 및 대입
3. **test03_comparison.amoro**: 비교 연산
4. **test04_logical.amoro**: 논리 연산
5. **test05_if_else.amoro**: 조건문
6. **test06_while.amoro**: 반복문
7. **test07_factorial.amoro**: 팩토리얼 계산 (반복문 활용)
8. **test08_functions.amoro**: 함수 정의 및 호출
9. **test09_fibonacci.amoro**: 피보나치 수열 (재귀 함수)
10. **test10_nested_loops.amoro**: 중첩 반복문
11. **test11_string_operations.amoro**: 문자열 처리
12. **test12_complex_expression.amoro**: 복잡한 수식 (우선순위)
13. **test13_modulo.amoro**: 나머지 연산
14. **test14_scope.amoro**: 변수 스코프
15. **test15_function_with_condition.amoro**: 함수 내 조건문

### 5.2 테스트 실행 방법

```bash
# 개별 테스트 실행
python -m compiler.main tests/test01_basic_arithmetic.amoro

# 모든 테스트 실행 (Windows)
for %f in (tests\*.amoro) do python -m compiler.main %f

# 모든 테스트 실행 (Linux/Mac)
for f in tests/*.amoro; do python -m compiler.main "$f"; done
```

## 6. 구현된 기능 및 제한 사항

### 6.1 구현된 기능

✅ **필수 기능 (모두 구현)**:
- 변수 선언 및 대입
- 산술/비교/논리 연산
- 조건문 (if/if-else)
- 반복문 (while)
- 함수 정의 및 호출

✅ **추가 기능**:
- 문자열 리터럴
- 주석 처리
- 변수 스코프 관리
- 클로저 지원 (함수 내부에서 외부 변수 접근)
- 재귀 함수 호출
- 내장 함수 (print)

### 6.2 미구현/제한 사항

❌ **미구현 기능**:
- 배열/리스트 자료구조
- 클래스/객체 지향 프로그래밍
- 문자열 연결 연산 (`+` 연산자로 문자열 연결 미지원)
- for 루프
- switch/case 문
- break/continue 문
- 예외 처리 (try-catch)
- 모듈 시스템 (import/export)
- 파일 입출력

⚠️ **제한 사항**:
- 부동소수점 연산의 정밀도 제한 (Python의 float 정밀도에 의존)
- 재귀 함수 호출 깊이 제한 없음 (스택 오버플로우 가능)
- 큰 숫자 연산 시 성능 저하 가능
- 문자열은 리터럴만 지원 (연결, 슬라이싱 등 미지원)

### 6.3 향후 개선 방향

1. **자료구조 추가**: 배열, 딕셔너리 지원
2. **타입 시스템 강화**: 정적 타입 검사 옵션
3. **성능 최적화**: 바이트코드 컴파일, JIT 컴파일
4. **표준 라이브러리**: 수학 함수, 문자열 처리 함수 등
5. **디버깅 도구**: 변수 추적, 브레이크포인트 등

## 7. 사용 예제

### 7.1 기본 예제

```amoro
// 변수 선언 및 산술 연산
let a = 10;
let b = 20;
let sum = a + b;
print sum;  // 30
```

### 7.2 조건문 예제

```amoro
let score = 85;
if (score >= 90) {
    print "A";
} else if (score >= 80) {
    print "B";
} else {
    print "C";
}
```

### 7.3 반복문 예제

```amoro
// 1부터 10까지 합계
let sum = 0;
let i = 1;
while (i <= 10) {
    sum = sum + i;
    i = i + 1;
}
print sum;  // 55
```

### 7.4 함수 예제

```amoro
// 최대값 구하기
function max(a, b) {
    if (a > b) {
        return a;
    } else {
        return b;
    }
}

let result = max(10, 20);
print result;  // 20
```

### 7.5 재귀 함수 예제

```amoro
// 팩토리얼 계산
function factorial(n) {
    if (n <= 1) {
        return 1;
    }
    return n * factorial(n - 1);
}

print factorial(5);  // 120
```

## 8. 결론

이 프로젝트는 컴파일러의 기본 구조인 어휘 분석, 구문 분석, 실행 단계를 모두 구현한 완전한 인터프리터입니다. 교육 목적에 맞게 코드를 명확하게 작성하고, 각 단계를 모듈화하여 이해하기 쉽게 구성했습니다.

이 언어는 간단하지만 필수적인 프로그래밍 개념을 모두 포함하고 있어, 컴파일러 이론 학습에 적합한 프로젝트입니다.

