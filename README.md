# 간단한 프로그래밍 언어 컴파일러/인터프리터

이 프로젝트는 교육용으로 설계된 간단한 프로그래밍 언어의 인터프리터 구현입니다.

## 프로젝트 개요

이 프로젝트는 컴파일러 과제를 위해 개발된 간단한 프로그래밍 언어의 전체 컴파일러 파이프라인을 구현합니다:
- **어휘 분석 (Lexer)**: 소스 코드를 토큰으로 분해
- **구문 분석 (Parser)**: 토큰 스트림을 AST로 변환
- **인터프리터 (Interpreter)**: AST를 실행하여 프로그램 평가

## 언어 기능

이 언어는 다음 기능을 지원합니다:

- ✅ 변수 선언 및 대입 (`let`, `=`)
- ✅ 산술 연산 (`+`, `-`, `*`, `/`, `%`)
- ✅ 비교 연산 (`==`, `!=`, `<`, `<=`, `>`, `>=`)
- ✅ 논리 연산 (`&&`, `||`, `!`)
- ✅ 조건문 (`if` / `if-else`)
- ✅ 반복문 (`while`)
- ✅ 함수 정의 및 호출 (`function`, `return`)
- ✅ 출력문 (`print`)

## 의존성

- Python 3.7 이상

별도의 외부 라이브러리 의존성은 없습니다. Python 표준 라이브러리만 사용합니다.

## 빌드 방법

이 프로젝트는 Python으로 작성되어 있어 별도의 빌드 과정이 필요하지 않습니다.

## 실행 방법

### 1. 파일 실행

```bash
python -m compiler.main <파일경로>
```

예시:
```bash
python -m compiler.main examples/hello.lang
```

### 2. 대화형 모드 (REPL)

```bash
python -m compiler.main
```

대화형 모드에서 코드를 입력하고 실행할 수 있습니다. 종료하려면 `exit` 또는 `quit`를 입력하세요.

### 3. 도움말

```bash
python -m compiler.main --help
```

## 프로젝트 구조

```
compiler_ass2/
├── __init__.py          # 패키지 초기화
├── lexer.py             # 어휘 분석기
├── parser.py            # 구문 분석기
├── ast.py               # AST 노드 정의
├── interpreter.py       # 인터프리터
├── main.py              # 메인 실행 파일
├── project_documentation.md  # 프로젝트 설명서
└── README.md            # 이 파일
```

## 예제 프로그램

### 기본 예제

```lang
// 변수 선언 및 출력
let x = 10;
print x;

// 산술 연산
let y = x + 5;
print y;

// 조건문
if (x > 5) {
    print "x is greater than 5";
} else {
    print "x is not greater than 5";
}

// 반복문
let i = 0;
while (i < 5) {
    print i;
    i = i + 1;
}

// 함수 정의 및 호출
function add(a, b) {
    return a + b;
}

let result = add(3, 4);
print result;
```

더 많은 예제는 `examples/` 디렉토리를 참조하세요.

## 문법 요약

### 변수 선언
```
let <식별자> [= <표현식>];
```

### 변수 대입
```
<식별자> = <표현식>;
```

### 조건문
```
if (<조건>) {
    <문장들>
} [else {
    <문장들>
}]
```

### 반복문
```
while (<조건>) {
    <문장들>
}
```

### 함수 정의
```
function <함수명>(<매개변수1>, <매개변수2>, ...) {
    <문장들>
    [return <표현식>;]
}
```

### 함수 호출
```
<함수명>(<인자1>, <인자2>, ...);
```

자세한 문법 정의는 `project_documentation.md`를 참조하세요.

## 에러 처리

프로그램 실행 중 발생할 수 있는 에러:

- **ParseError**: 구문 오류 (예: 괄호 불일치, 예상치 못한 토큰)
- **RuntimeError**: 런타임 오류 (예: 정의되지 않은 변수, 0으로 나누기)

에러 메시지는 파일명, 줄 번호, 열 번호와 함께 출력됩니다.

## 제한 사항

- 배열/리스트 미지원
- 클래스/객체 지향 프로그래밍 미지원
- 타입 시스템 없음 (동적 타입)
- 문자열 연결 연산자 없음 (숫자만 `+` 연산 가능)
- 재귀 함수 호출 제한 (스택 오버플로우 가능)

자세한 제한 사항은 `project_documentation.md`를 참조하세요.


