# 1주차

### 1장. 들어가며

- 웹의 다양한 컨텐츠를 표현하는 새로운 언어의 필요성에 의해 등장한 자바스크립트
- 모든 브라우저에서 동일하게 동작하도록 ESMAScript 탄생
- 웹 사이트 vs 웹 애플리케이션
    - 웹 사이트 : 정적인 웹, 사용자와의 상호작용 없는 단방향 정보 제공
    - 웹 애플리케이션 : 사용자와 상호작용하는 쌍방향 소통 웹
- 거대해진 웹 애플리케이션 개발을 위해 컴포넌트 베이스 개발 방법론이 등장, 협업의 필요성이 증가하며 자바스크립트 한계점에 보완이 필요해짐
    - 자바스크립트 한계점
        - 동적 타입 언어
        - 동적 타입핑 시스템으로 인해 개발자의 의도와 다른 코드 결과물이 나오기도 함
- 자바스크립트 한계 극복을 위한 여러 해결 방안이 제시되었지만 그것에도 단점이 존재했고, 해당 단점까지 보완하는 타입스크립트가 등장
- 타입스크립트
    - 안전성 보장 : 컴파일 단계에서 타입 검사
    - 개발 생산성 향상
    - 협업에 유리
    - 자바스크립트에 점진적으로 적용 가능

### 2장. 타입이란

- 타입 : 값이 가질 수 있는 유효한 범위의 집합
- 개발자는 타입을 사용해서 값의 종류를 명시할 수 있고 메모리를 더욱 효율적으로 사용할 수 있음.
- 정적 타입 vs 동적 타입
    - 정적 타입 : 모든 변수의 타입이 컴파일 타임에 결정되어 런타임 이전에 에러를 발견할 수 있는 안전성 보장
    - 동적 타입 : 변수 타입이 런타임에 결정되어 개발자는 직접 타입을 정의해줄 필요가 없음
- 타입 안전성 : 프로그램이 유효하지 않은 작업을 수행하지 않도록 방지
- 타입 스크립트의 컴파일 결과물은 자바스크립트 파일 ⇒ 템플릿 언어, 확장 언어로 해석하기도 함
- 타입 애너테이션 : 타입을 명시적으로 선언해서 어떤 타입 값이 저장된 것인지를 컴파일러에 직접 알려주는 문법으로 언어마다 방법이 다름
- 타입 스크립트는 구조로 타입을 구분하는 구조적 타이핑 방법
    - 구조적 서브 타이핑 : 객체가 가지고 있는 속성을 바탕으로 타입을 구분하기에 이름이 다른 객체라도 속성이 동일하다면 호환이 가능
    
    ```tsx
    // 여러 타입을 동시에 가질 수 있음
    type stringOrNumber = string | number;
    ```
    
    ```tsx
    interface Pet {
      name: string
    }
    
    interface Cat {
      name: string
      age: number
    }
    
    let pet: Pet;
    let cat: Cat = { name: "Zag", age: 2 };
    
    // ✅ OK
    pet = cat;
    ```
    
    - 하나의 타입이 다른 타입의 서브 타입이라면 전자의 인스턴스는 후자의 타입이 필요한 곳에 언제든지 위치할 수 있음.
    
    ```tsx
    class Person {
      name: string;
    
      age: number;
    
      constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
      }
    }
    
    class Developer {
      name: string;
    
      age: number;
    
      sleepTime: number;
    
      constructor(name: string, age: number, sleepTime: number) {
        this.name = name;
        this.age = age;
        this.sleepTime = sleepTime;
      }
    }
    
    function greet(p: Person) {
      console.log(`Hello, I'm ${p.name}`);
    }
    
    const developer = new Developer("zig", 20, 7);
    
    greet(developer); // Hello, I'm zig
    ```
    
    - 서로 다른 두 타입 간의 호환성은 타입 내부 구조에 의해 결정
    - 자바스크립트는 덕 타이핑 기반
        - 덕 타이핑 : 어떤 함수의 매개변숫값이 올바르게 주어진다면 그 값이 어떻게 만들어졌는지는 신경쓰지 않고 사용
    - 타입스크립트는 자바스크립트트의 덕타이핑 기반 특징을 수용하여 명시적인 이름으로 구분하는 것이 아닌 객체나 함수가 가진 구조적 특징을 기반으로 타이핑하는 방식 채택 ⇒ 유연한 타이핑 가능해짐.
    
    ```tsx
    interface Cube {
      width: number
      height: number
      depth: number
    }
    
    function addLines(c: Cube) {
      let total = 0;
    
      for (const axis of Object.keys(c)) {
        // 🚨 Element implicitly has an 'any' type
        // because expression of type 'string' can't be used to index type 'Cube'.
        // 🚨 No index signature with a parameter of type 'string'
        // was found on type 'Cube'
        const length = c[axis];
    
        total += length;
      }
    }
    
    const namedCube = {
      width: 6,
      height: 5,
      depth: 4,
      name: "SweetCube", // string 타입의 추가 속성이 정의되었다
    };
    
    addLines(namedCube); // ✅ OK
    ```
    
    - 그러나 유연한 타이핑으로 인해 위 코드처럼 추가 속성으로 인한 문제가 생기기도 하여 이에 대한 해결 방법으로 유니온 방법이 생겨남.
    - 타입스크립트는 점진적으로 타입을 확인한다. 컴파일 타임에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용
        - 타입을 지정한 경우 정적으로 검사
        - 지정하지 않은 경우 동적으로 검사 (암시적 타입 변환 발생)
    - 타입스크립트에서 프로그램을 컴파일 하는데 모든 타입을 알아야 하는 것은 아니지만 타입 스크립트는 컴파일 타임에 프로그램의 모든 타입을 알고 있을 때, 최상의 결과를 보여줌.

### 질문

- 타입스크립트는 정적 언어인가요? 동적 언어인가요? 이유와 함께 답해주세요.
- 타입스크립트 사용 시, 타입을 선언하지 않은 경우 어떤 일이 벌어지나요?
