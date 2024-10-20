## 6장 
### 6.1 자바스크립트의 런타임과 타입스크립트의 컴파일
1. 런타임과 컴파일타임

자바스크립트는 대표적인 고수준 언어에 속하며 컴파일러나 인터프리터에 의해 저수준 언어(기계가 이해할 수 있음)로 번역되어 실행됩니다.

- 컴파일타임: 개발자가 작성한 소스코드가 컴파일러에 의해 기계어 코드로 변환되어 실행이 가능한 프로그램이 되는 단계
- 런타임: 컴파일 과정을 마친 응용 프로그램이 메모리에 적재되어 실행되는 시간

2. 자바스크립트 런타임

자바스크립트 런타임은 자바스크립트가 실행되는 환경을 의미합니다. 대표적인 자바스크립트 런타임으로 크롬이나 사파리 같은 `인터넷 브라우저`와 `Node.js`가 있습니다.

자바스크립트 런타임의 주요 구성요소는 `자바스크립트 엔진`, `웹 API`, `콜백 큐`, `이벤트 루프`, `렌더 큐`가 있습니다.

다음과 같이 에러가 발생하는 이유는 런타임에서 자바스크립트 에러가 발생하기 때문입니다.

```js
let foo;
foo.bar; // TypeError: Cannot read properties of undefined (reading 'bar')
```

3. 타입스크립트의 컴파일

타입스크립트는 tsc라고 불리는 컴파일러를 통해 자바스크립트 코드로 변환됩니다. 

타입스크립트 컴파일러는 소스코드를 해석하여 `AST(최소 구문 트리)`를 만들고 이후 타입 확인을 거친 다음에 결과 코드를 생성합니다.

**타입스크립트 컴파일러가 소스코드를 컴파일하여 프로그램이 실행되기까지의 과정**

1. 타입스크립트 소스 코드를 타입스크립트 AST로 만든다. (tsc)
2. 타입 검사기가 AST를 확인하여 타입을 확인한다. (tsc)
3. 타입스크립트 AST를 자바스크립트 소스로 변환한다. (tsc)
4. 자바스크립트 소스코드를 자바스크립트 AST로 만든다. (런타임)
5. AST가 바이트 코드로 변환된다. (런타임)
6. 런타임에서 바이트 코드가 평가되어 프로그램이 실행된다. (런타임)

이때 타입스크립트 소스코드의 타입은 1~2 단계에서만 사용되며 3단계부터는 타입을 확인하지 않습니다.

즉, 개발자가 작성한 타입 정보는 단지 타입을 확인하는 데만 쓰이며, 최종적으로 만들어지는 프로그램에는 아무런 영향을 주지 않습니다.

### 6.2 타입스크립트 컴파일러의 동작
1. 코드 검사기로서의 타입스크립트 컴파일러

타입스크립트 컴파일러는 코드에 타입 오류가 없는지를 확인하고 문법 에러와 타입 관련 에러를 모두 검출합니다.

```js
const developer = {
    work(){
        console.log("working...")
    }
}

developer.working();
developer.sleep(); // TypeError: developer.sleep is not a function
```

위 코드를 자바스크립트로 작성하는 시점에서는 에러가 발생하지 않지만 실행을 하면 에러가 발생합니다. 같은 코드를 타입스크립트로 작성해보겠습니다.

```ts
const developer = {
    work(){
        console.log("working...")
    }
}

developer.working();
developer.sleep(); // Property 'sleep' does not exist on type '{ work(): void; }'
```

타입스크립트는 코드를 실행하기 전에 에러를 사전에 발견하여 알려줍니다. 그리고 표시되는 에러 메시지도 달라집니다.

타입스크립트 컴파일러는 tsc binder를 사용하여 타입 검사를 하며, 컴파일타임에 타입 오류를 발견합니다.
타입 검사를 거쳐 코드를 안전하게 만든 이후에는 타입스크립트 AST를 자바스크립트 코드로 변환합니다.

2. 코드 변환기로서의 타입스크립트 컴파일러

타입스크립트 소스코드는 브라우저와 같은 런타임에서 실행될 수 없습니다. 소스코드를 파싱하고 자바스크립트 코드로 변환해야 비로소 실행할 수 있게 됩니다.

타입스크립트 컴파일러는 타입을 검사한 다음에 코드를 구버전의 자바스크립트로 트랜스파일합니다.

다음 예시는 타입스크립트 코드를 자바스크립트 코드로 변환한 결과를 보여줍니다. 타입스크립트 컴파일러의 target 옵션을 사용하여 특정 버전의 자바스크립트로 변환할 수 있습니다.

```ts
// target: ES5
type Fruit = 'banana' | 'watermelon' | 'orange' | 'apple' | 'kiwi' | 'mango';
const fruitBox: Fruit[] = ['banana', 'apple', 'mango'];

const welcome = (name: string) => {
    console.log(`hi! ${name}`);
}
```

```js
"use strict";
var fruitBox = ['banana', 'apple', 'mango'];
var welcome = function (name) {
    console.log("hi!".concat(name));
};
```

타입스크립트 컴파일러는 타입 검사를 수행한 후 코드 변환을 시작하는데, 이때 타입 오류가 있더라도 일단 컴파일을 진행합니다. 변환 과정은 타입 검사와 독립적으로 동작하기 때문입니다.

타입스크립트 컴파일 이후에는 타입이 제거되어 순수한 자바스크립트 코드만 남습니다. 이때 주의해야 하는 경우가 있는데요.

```ts
interface Square {
    width: number;
}

interface Rectangle extends Square {
    height: number;
}


type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
    if (shape instanceof Rectangle) {
    // ‘Rectangle’ only refers to a type, but is being used as a value here
    // Property ‘height’ does not exist on type ‘Shape’
    // Property ‘height’ does not exist on type ‘Square’
        return shape.width * shape.height;
    } else {
    return shape.width * shape.width;
    }
}
```

Rectangle은 이미 컴파일 과정에서 제거되었기 때문에 런타임에서 사용할 수가 없어 위같은 에러가 발생합니다.

> 
> **타입스크립트 컴파일러와 바벨의 차이점**
> 
> 타입스크립트는 타입 검사를 수행하고 코드를 변환하지만 바벨은 코드 변환만 수행


### 6.3 타입스크립트 컴파일러의 구조
실제 타입스크립트 컴파일러의 구성 요소를 훑어보며 타입스크립트 컴파일러의 동작 방식을 이해해보겠습니다.

컴파일러는 하나의 프로그램으로 이를 구현한 소스 파일이 존재합니다. 타입스크립트 공식 깃허브에서 [소스 파일](https://github.com/microsoft/TypeScript/tree/main/src/compiler)을 확인할 수 있습니다.

타입스크립트 컴파일러는 다섯 단계를 거쳐 타입 검사와 자바스크립트 소스 변환을 진행합니다.
1. 스캐너: .ts 토큰화
2. 파서: 토큰 기반 AST 생성
3. 바인더: AST 노드 기반 심볼 생성
4. 체커: AST + 심볼 기반 타입 검사
5. 이미터: AST + 코드 검사 기반 .js 생성
<br/>

1. 프로그램

타입스크립트 컴파일러는 tsc 명령어로 실행됩니다. 컴파일러는 tsconfig.json에 명시된 컴파일 옵션을 기반으로 컴파일을 수행합니다.

먼저 전체적인 컴파일 과정을 관리하는 프로그램 객체(인스턴스)가 생성됩니다. 

이 프로그램 객체는 컴파일할 타입스크립트 소스 파일과 소스 파일 내에서 임포트된 파일을 불러오는데, 가장 최초로 불러온 파일을 기준으로 컴파일 과정이 시작됩니다.

2. 스캐너

스캐너는 타입스크립트 소스 파일을 어휘적으로 분석하여 토큰을 생성하는 역할을 합니다. 다시말해 소스코드를 작은 단위로 나누어 의미 있는 토큰으로 변환하는 작업을 수행합니다.

```js
const woowa = 'bros'
```

위의 변수를 선언하는 코드는 [해당 링크](https://www.typescriptlang.org/play/?ssl=1&ssc=6&pln=1&pc=7&install-plugin=playground-ts-scanner#code/MYewdgzgLgBA7iEcCGMC8MBEAjATiCTIA)에서 분석 결과를 확인할 수 있습니다.

3. 파서

스캐너가 소스 파일을 토큰으로 나눠주면 파서는 그 토큰 정보를 이용하여 AST를 생성합니다.

AST는 컴파일러가 동작하는 데 핵심 기반이 되는 자료 구조로, 소스코드의 구조를 트리 형태로 표현합니다. AST의 최상위 노드는 타입스크립트 소스 파일이며, 최하위 노드는 파일의 끝 지점으로 구성됩니다.

스캐너는 `어휘적 분석`을 통해 토큰 단위로 소스코드를 나누지만 파서는 이렇게 생성된 토큰 목록을 활용하여 `구문적 분석`을 수행합니다.

이를 통해 코드의 실질적인 구조를 노드 단위의 트리 형태로 표현합니다. 각각의 노드는 코드상의 위치, 구문 종류, 코드 내용과 같은 정보를 담고 있습니다. 

예를 들어 ( ) 에 해당하는 토큰이 있을 때 파서가 AST를 생성하는 과정에서 이 토큰이 실질적으로 함수의 호출인지, 함수의 인자인지 또는 그룹 연산자인지가 결정됩니다.

```js
function normalFunction() {
    console.log("normalFunction");
}

normalFunction();
```

앞의 코드는 [해당 링크](https://ts-ast-viewer.com/#code/GYVwdgxgLglg9mABGOAnAtgQwDYDFzTxgAUAlAN4BQiNiECAznNgKYB02cA5sQEQoYc+SLAS9SAbkoBfSpQFY8BUSUlA)에서 변환 결과를 확인할 수 있습니다. 

4. 바인더

바인더의 주요 역할은 체커 단계에서 타입 검사를 할 수 있도록 기반을 마련하는 것입니다. 바인더는 타입 검사를 위해 심볼이라는 데이터 구조를 생성합니다.

심볼은 이전 단계의 AST에서 선언된 타입의 노드 정보를 저장합니다. 심볼의 인터페이스 일부는 다음과 같이 구성됩니다.

```ts
export interface Symbol {
    flags: SymbolFlags; // Symbol flags

    escapedName: string; // Name of symbol

    declarations?: Declaration[]; // Declarations associated with this symbol

    // 이하 생략...
}
```

`flags` 필드는 AST에서 선언된 타입의 노드 정보를 저장하는 식별자입니다. 심볼을 구분하는 식별자 목록은 다음과 같습니다.

```ts
// src/compiler/types.ts
export const enum SymbolFlags {
    None = 0,
    FunctionScopedVariable = 1 << 0, // Variable (var) or parameter
    BlockScopedVariable = 1 << 1, // A block-scoped variable (let or const)
    Property = 1 << 2, // Property or enum member
    EnumMember = 1 << 3, // Enum member
    Function = 1 << 4, // Function
    Class = 1 << 5, // Class
    Interface = 1 << 6, // Interface
    // ...
}
```

심볼 인터페이스의 `declarations` 필드는 AST 노드의 배열 형태를 보입니다. 결과적으로 바인더는 심볼을 생성하고 해당 심볼과 그에 대응하는 AST 노드를 연결하는 역할을 수행합니다.

```ts
type SomeType = string | number;

interface SomeInterface {
    name: string;
    age?: number;
}

let foo: string = "LET";

const obj = {
name: "이름",
age: 10,
};
class MyClass {
name;
age;
constructor(name: string, age?: number) {
this.name = name;
this.age = age ?? 0;
}
}

const arrowFunction = () => {};

function normalFunction() { }

arrowFunction();

normalFunction();

const colin = new MyClass("colin");
```

앞의 코드는 [해당 링크](https://www.typescriptlang.org/play/?&install-plugin=playground-ts-symbols#code/C4TwDgpgBAyg9gWwgFXNAvFAzsATgSwDsBzKAHykIFcEAjCXAbgChmjgGAzAQwGNp4SAJKEOuHvygBvZlDmVuSAFzY8RYi3lRuxCAH4V1OgxYBfVgBsIwKJzhwVOAiSiYARABkAosjctmvHCEOFBwtABWrtLMhIoQKm6ALuOAIZ1uADTMOvFQAIwADBmmLLwW3FhYUACyIADCpeXRsUgsWcVBTlS8wHC4ABRN2U7qadq6BpQ09LgAlNHAABb4WAB0A1EDLAtLy1lRu3p6UHlmzOYB7TbcuLhwAO4AYlSEXfhBUb2z6AB80kWsnE8Xm9CD0ENwLI9nsBXoQPtIoGcrjcHoDoUEPv4QbgwRDUTCMaxAsEbIELER1hBblVavUsL03KSiG5powgA)에서 심볼 결과를 확인할 수 있습니다.
근데 동작을 안하네요 저는.. 책에 나온 내용을 적도록 하겠습니다.

```text
Some Type -> TypeAliasDeclaration
Some Interface -> InterfaceDeclaration
foo -> VariableDeclaration
obj -> VariableDeclaration
MyClass -> ClassDeclaration
arrowFunction -> VariableDeclaration
colin -> VariableDeclaration
```

5. 체커와 이미터

체커는 파서가 생성한 AST와 바인더가 생성한 심볼을 활용하여 타입 검사를 수행합니다. 이 단계에서 체커의 소스 크기는 4.7.3 버전 기준 약 2.7MB 정도로 이전 단계의 파서의 소스 크기보다 매우 큽니다.
전체 컴파일 과정에서 타입 검사가 차지하는 비중이 크다는 것을 짐작할 수 있습니다. 

체커의 주요 역할은 AST의 노드를 탐색하면서 심볼 정보를 불러와 주어진 소스 파일에 대해 타입 검사를 진행하는 것입니다. 

체커의 타입 검사는 다음 컴파일 단계인 이미터에서 실행됩니다. `checker.ts`의 `getDiagnostics()` 함수를 사용해서 타입을 검증하고 타입 에러에 대한 정보를 보여줄 메세지를 저장합니다. 

이미터는 타입스크립트 소스 파일을 변환하는 역할을 합니다. 즉, 타입스크립트 소스를 js 파일과 타입 선언 파일(d.ts)로 생성합니다.

이미터는 타입스크립트 소스 파일을 변환하는 과정에서 개발자가 설정한 타입스크립트 설정 파일을 읽어오고, 체커를 통해 코드에 대한 타입 검증 정보를 가져옵니다.
그리고 `emitter.ts` 파일 내부의 `emitFiles()` 함수를 사용하여 타입스크립트 소스 변환을 진행합니다.


지금까지 살펴본 타입스크립트의 컴파일 과정을 정리하면 다음과 같습니다.
1. tsc 명령어를 실행하여 프로그램 객체가 컴파일 과정을 시작한다.
2. 스캐너는 소스 파일을 토큰 단위로 분리한다.
3. 파서는 토큰을 이용하여 AST를 생성한다.
4. 바인더는 AST의 각 노드에 대응하는 심볼을 생성한다. 심볼은 선언된 타입의 노드 정보를 담고 있다.
5. 체커는 AST를 탐색하면서 심볼 정보를 활용하여 타입 검사를 수행한다.
6. 타입 검사 결과 에러가 없다면 이미터를 사용해서 자바스크립트 소스 파일로 변환한다.
