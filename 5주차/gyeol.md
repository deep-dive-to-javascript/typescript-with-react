# 6장 - 타입스크립트 컴파일

## 6.1 자바스크립트의 런타임과 타입스크립트의 컴파일

### 1. 런타임과 컴파일타임

- 컴파일타임
    - 컴파일러에 의해 소스코드가 기계어 코드로 변환되어 실행 가능한 프로그램이 되는 단계
- 런타임
    - 컴파일이 완료된 후 프로그램이 메모리에 적재되어 실행되는 시간

### 2. 자바스크립트 런타임

- 자바스크립트가 실행되는 환경
- 다양한 종류와 버전이 있으며 각 런타임은 자바스크립트 코드를 실행하는 환경과 규칙을 정
- 대표적인 예
    - 크롬, 사파리와 같은 인터넷 브라우저, Nose.js, React Native, Electron
- 자바스크립트 런타임 구성요소
    - 자바스크립트 엔진, 웹 API, 콜백큐, 이벤트 루프, 렌더 큐

### 3. 타입스크립트의 컴파일

- 타입스크립트는 tsc라고 불리는 컴파일러를 통해 자바스크립트 코드로 변환
- 고수준 언어(TS) → 고수준 언어(JS)로 변환되는 것이므로  트랜스파일이라고 부르기도함
- 타입스크립트 컴파일러가 소스코드를 컴파일하여 프로그램이 실행되기까지의 과정
    1. 타입스크립트 소스코드를 타입스크립트 AST로 만듦(tsc)
    2. 타입 검사기가 AST를 확인하여 타입을 확인(tsc)
    3. 타입스크립트 AST를 자바스크립트 소스로 변환(tsc)
    4. 자바스크립트 소스코드를 자바스크립트 AST로 만듦(런타임)
    5. AST가 바이트코드로 변환(런타임)
    6. 런타임에서 바이트코드가 평가되어 프로그램이 실행됨(런타임)

> **✅ AST(Absctract Syntax Tree)**
> <br />
> 컴파일러가 소스코드를 해석하는 과정에서 생성된 데이터 구조
> 컴파일러는 어휘적 분적과 구문 분석을 통해 소스코드를 노트 단위의 트리 구조로 구성함
> 
- 타입스크립트 소스코드의 타입은 **1~2단계에서만 사용**되며, 3단계부터는 타입을 확인하지않음
- 타입스크립트는 컴파일타임에 타입을 검사하므로 에러가 발생하면 프로그램이 실행되지 않음
 → 같은 이유로 타입스크립트를 `정적타입검사기`라고 부름

## 6.2 타입스크립트 컴파일러의 동작

### 1. 코드 검사기로서의 타입스크립트 컴파일러

- 타입스크립트에서는 컴파일타임에 코드 타입을 확인하므로 코드를 실행하지 않고도 오류를 발견할 수 있음
- 또한 정적으로 코드를 분석하여 에러를 검출함. 코드 실행 전 자바스크립트 런타임에서 발생할 수 있는 에러를 사전에 알려줌
    
    → 컴파일타임에 문법 에러와 타입 관련 에러를 모두 검출
    
- 타입스크립트 컴파일러는 tsc binder를 통해 타입을 검사하고 컴파일타임에 타입 오류를 발견함
- 타입 검사를 거쳐 코드를 안전하게 만든 후 타입스크립트 AST를 자바스크립트 코드로 변환함
    
    > 📌 **ts binder**
    > <br />
    > 바인더는 트리를 돌며 트리의 각 선언을 방문합니다. 그리고 찾은 각 선언에 대해 해당 위치와 선언 종류를 기록하는  `Symbol` 을 생성합니다. 그런 다음 현재 범위인 함수, 블록 또는 모듈 파일과 같은 포함 노드의  `SymbolTable`에 해당  `Symbol` 을 저장합니다.  `Symbols`는 타입 체커가 이름을 찾아보고 해당 선언을 확인하여 타입을 결정할 수 있게 해줍니다. 또한 어떤 종류의 정의인지에 대한 간단한 요약도 포함합니다. (주로 value, type, namespace 여부)
    > 
    
    [Codebase Compiler Binder](https://github.com/microsoft/TypeScript/wiki/Codebase-Compiler-Binder)
    

### 2. 코드 변환기로서의 타입스크립트 컴파일러

- 타입스크립트 컴파일러는 타입을 검사한 후 타입스크립트 코드를 각자의 런타임 환경에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일함
- 타입스크립트 소스코드를 파싱하고 자바스크립트 코드로 변환해야 런타임에서 실행이 가능해짐
- 타입스크립트 컴파일러는 타입 검사를 진행할 때 타입 오류가 있더라도 일단 컴파일을 진행함
→ **타입스크립트 코드가 자바스크립트 코드로 변환되는 과정은 타입 검사와 독립적으로 동작함**

📌 타입스크립트 컴파일러의 역할과 바벨

- 바벨(Babel)
    - ECMAScript 2015 이후의 코드를 현재 또는 오래된 브라우저와 호환되는 버전으로 변환해주는 자바스크립트 컴파일러
- tsc와 바벨의 공통점
    - 소스코드를 ES5 이하의 자바스크립트 코드로 컴파일함
- tsc와 바벨의 차이점
    - tsc와 달리 바벨은 타입 검사를 진행하지않음

## 6.3 타입스크립트 컴파일러의 구조

- 컴파일러는 하나의 프로그램으로 이를 구현한 소스파일이 존재함
    
    https://github.com/microsoft/TypeScript/tree/main/src/compiler
    
- 타입스크립트 컴파일러는 아래의 단계를 거쳐 타입 검사와 자바스크립트 소스 변환을 진행
    ![image](https://github.com/user-attachments/assets/d3e62ce1-3b27-4115-9661-c905e8cc4f16)
    
- 타입스크립트 컴파일러가 동작하는 방식
    1. tsconfig 읽기 : 타입스크립트 프로젝트라면, root에 **`tsconifg.json`** 을 읽는 작업부터 시작
    2. preprocess : 파일의 root 부터 시작해서 imports로 연결된 가능한 모든 파일을 찾음
    3. tokenize & parse : **`.ts`** 로 작성된 파일을 신택스 트리로 변경
    4. binder: 3번에서 변경한 신택스 트리를 기준으로, 해당 트리에 있는 symbol (**`const`** 등) 을 identifier로 변경
    5. 타입체크 : binder와 신택스 트리를 기준으로 타입 체크
    6. transform : 신택스트리를 1번에서 읽었던 옵션에 맞게 변경
    7. emit: 신택스 트리를 **`.js`** **`.d.ts`** 파일 등으로 변경
    
    ✔ 1~3번 : 소스 코드를 읽어 데이터로 만드는 과정
    
    ✔ 4~5번 : 타입 체킹 과정
    
    ✔ 6~7번 : 파일을 만드는 과정
    
    [타입스크립트 컴파일러는 어떻게 동작하는가?](https://yceffort.kr/2022/05/how-typescript-compiler-works)
    

### 1. 프로그램(Program)

- 타입스크립트 컴파일러는 tsc 명령어로 실행함
- 컴파일러는 tsconfig.json에 명시된 컴파일 옵션을 기반으로 컴파일을 수행함
- 먼저 전체적인 컴파일 과정을 관리하는 프로그램 객체(인스턴스)가 생성됨
- 프로그램 객체가 컴파일할 타입스크립트 소스파일과 소스 파일 내에 임포트된 파일을 불러옴(위 과정 2번에 해당)

### 2. 스캐너(Scanner)

- **타입스크립트 소스 파일을 어휘적으로 분석하여 토큰을 생성하는 역할**
    
    ```tsx
    const message: string = "Hello, world!"
    ```
    
    - Scan results(TS Playground에서 scanner 플러그인을 이용하면 확인 가능)
        - `ConstKeyword`
        - `WhitespaceTrivia`
        - `Identifier`
        - `ColonToken`
        - `WhitespaceTrivia`
        - `StringKeyword`
        - `WhitespaceTrivia`
        - `EqualsToken`
        - `WhitespaceTrivia`
        - `StringLiteral`
- 파일을 처음부터 읽어가며 특정 키워드 내지는 예약어가 있는지, identifier가 있는지 등 순차적으로 확인
- Scanner는 이 과정에서 코드 문자열의 정합성도 검사
- 실제 Scanner.ts 코드
    
    https://github.com/microsoft/TypeScript/blob/main/src/compiler/scanner.ts
    

### 3. Parser

- 스캐너가 소스 파일을 토큰으로 나눠주면 파서는 그 토큰 정보를 이용해서 AST를 생성함
- AST(Abstract Syntax Tree, 추상 구문 트리)
    - 컴파일러가 동작하는데 핵심 기반이 되는 자료구조
    - 소스코드의 구조를 트리형태로 표현
    - AST의 최상위 노드는 타입스크립트 소스파일
    - AST의 최하위 노드는 파일의 끝 지점
- 예시
    
    ```tsx
    const message: string = 'Hello, world!'
    ```
    ![image](https://github.com/user-attachments/assets/a883bf8d-c2e6-451b-88c3-003bd4b4dff5)

    [TypeScript AST Viewer](https://ts-ast-viewer.com/#code/MYewdgzgLgBAtgUwhAhgcwQLhtATgSzDRgF4YByACQQBsaQAaGAdxFxoBMBCcoA)
    
    - Scanner가 만든 토큰을 기준으로 AST를 만듦
      
        ![image](https://github.com/user-attachments/assets/7af17db1-2683-4e7b-ad03-868f5abafe32)

        - **`VariableStatement`**: **`const`** 를 시작으로 한 변수 선언 구문을 의미
        - **`VariableDeclarationList`**: 여기에서 선언된 변수 배열을 나타냄
        배열인 이유 : let a, b, c = 3 과 같이 여러 변수를 한 구문에서 선언할 수 있기 때문
        - **`VariableDeclaration`**: **`message`** 선언부를 의미
        - **`Identifier`**: **`message`**
        - **`StringKeyword`**: **`string`** 타입 선언부
        - **`StringLiteral`**: **`Hello, world!'`**

### 4. 바인더(Binder)

- 체커 단계에서 타입 검사를 할 수 있도록 기반을 마련
- **타입 검사를 위해 심볼이라는 데이터 구조를 생성**
- 심볼(Symbol)
    - 이전 단계의 AST에서 선언된 타입의 노드 정보를 저장
    - 심볼의 인터페이스(일부)
        
        ```tsx
        export interface Symbol {
        	flag: SymbolFlags; // Symbol flags
        	escapedName: string; // Name of symbol
        	declarations?: Declaration[]; // Declartions associated with this symbol
        	// ...
        }
        ```
        
        - flags 필드 → AST에서 선언된 타입의 노드 정보를 저장하는 식별자
        - declarations 필드 → AST 노드의 배열 형태
    - 심볼을 구분하는 식별자 목록
        
        ```tsx
        // src/compiler/types.ts
        export const enum SymbolFlags {
        	None                    = 0,
        	FunctionScopedVariable  = 1 << 0,   // Variable (var) or parameter
        	BlockScopedVariable     = 1 << 1,   // A block-scoped variable (let or const)
        	Property                = 1 << 2,   // Property or enum member
        	EnumMember              = 1 << 3,   // Enum member
        	Function                = 1 << 4,   // Function
        	Class                   = 1 << 5,   // Class
        	Interface               = 1 << 6,   // Interface
        	// ... 
        }
        ```
        
- 바인더는 심볼을 생성하고 해당 심볼과 그에 대응하는 AST노드를 연결하는 역할

### 5. 체커(Checker)와 이미터(Emitter)

- **체커(Checker)**
    - 파서가 생성한 AST와 바인더가 생성한 심볼을 활용하여 타입 검사를 수행
    - AST의 노드를 탐색하면서 심볼 정보를 불러와 주어진 소스 파일에 대해 타입 검사를 진행
    - 대다수의 타입스크립트 validation이 이 단계에서 이루어짐
    - 대부분의 신택스 트리에는 이에 맞는 checker 함수가 존재
    → 이 신택스트리를 순회하면서 대부분의 객체들을 체크하게됨
 
        ![image](https://github.com/user-attachments/assets/bc6dbcb8-d220-43c4-9f0e-e71eba02bade)

    - 체커의 타입 검사는 다음 컴파일 단계인 이미터에서 실행됨
    - checker.ts의 `getDiagnostics()` 함수를 사용해서 타입을 검증하고 타입 에러에 대한 정보를 보여줄 에러메세지를 저장
    - checker.ts
        
        [raw.githubusercontent.com](https://raw.githubusercontent.com/microsoft/TypeScript/main/src/compiler/checker.ts)
        
- **이미터(Emitter)**
    - 타입스크립트 소스 파일을 변환하는 역할
    - 타입스크립트 소스(ts) → 자바스크립트 파일(js)과 타입 선언 파일(d.ts)로 생성
    - 타입스크립트 소스 파일을 변환하는 과정에서 개발자가 설정한 타입스크립트 설정 파일을 읽어오고 체커를 통해 코드에 대한 타임 검증 정보를 가져옴 
    그리고 emitter.ts 소스 파일 내부의 `EmitFiles()`함수를 통해 타입스크립트 소스 변환을 진행
