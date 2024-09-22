# 1회차 정리(1장~2장)

# 1장 - 들어가며

## 1.2 자바스크립트의 한계

### 1. 동적 타입 언어

- 자바스크립트는 동적 타입 언어
    - 변수에 타입을 명시적으로 지정하지 않음
    - 코드가 실행되는 런타임에 변숫값이 할당될 때 해당 값의 타입에 따라 변수 타입이 결정됨

### 2. 동적 타이핑 시스템의 한계

```jsx
// 이 함수는 숫자 a,b의 합을 반환한다
const sumNumber = (a, b) => {
	return a + b;
};

sumNumber(1, 2); // 3
sumNumber(100); // NaN
sumNumber('a', 'b'); // ab
```

- 코드에 적힌 주석과 함수 이름은 두 수의 합을 구하기 위한 함수라고 명시함
- 개발자의 의도와는 달리 하나의 숫자만 전달하거나 문자열을 전달하더라도 자바스크립트 엔진은 이 코드를 문제 없이 실행시켜버림
    
    → 자바스크립트 엔진에서 주석, 함수 이름, 개발자 의도는 고려대상이 아님
    

### 3. 한계 극복을 위한 해결 방안

- JSDoc
    - 모듈, 네임스페이스, 클래스, 메서드, 매개변수 등에 대한 API 문서 생성 도구
    - 주석의 성격을 지니기 때문에 강제성을 부여해 사용하기 어려움
- PropTypes
    - 리액트에서 컴포넌트 props의 타입을 검사하기 위해 사용하는 속성
    - prop에 유효한 값이 전달되었는지 확인 가능
    - 전체 애플리케이션의 타입 검사를 하는 데는 사용 불가능
    - 리액트에서만 사용할 수 있음
- 다트(Dart)
    - 구글이 자바스크립트를 대체하기 위해 제시한 새로운 언어
    - 타이핑 가능
    - 자바스크립트가 아닌 새로운 언어의 등장으로 평이 좋지 않았음

### 4. 타입스크립트의 등장

- 위와 같은 문제점을 해결하기 위해 마이크로소프트는 자바스크립트의 슈퍼셋 언어인 `타입스크립트`를 공개
    

    💡 **슈퍼셋(Superset)**
    <br />
    - 기존 언어에 새로운 기능과 문법을 추가해서 보완하거나 향상하는 것
    - 슈퍼셋 언어는 기존 언어와 호환되며 컴파일러 등을 통해 기존 언어 코드로 변환되어 실행됨
    

    
- 타입스크립트의 특징
    - 안정성 보장
        - 정적 타이핑을 제공
        - 컴파일 단계에서 타입 검사를 실행 → 타입에러를 줄이고 런타임 에러를 방지
    - 개발 생산성 향상
        - VSCode 등 IDE에서 타입 자동 완성 기능을 제공
            
            → 변수와 함수 타입을 추론 가능하여 개발 향상성이 향상됨
            
    - 협업에 유리
        - 인터페이스가 기술되면 코드를 더 쉽게 이해할 수 있게 도와줌
        → 복잡한 애플리케이션 개발, 협업에 유리함
    - 자바스크립트에 점진적으로 적용 가능

# 2장 - 타입

## 2.1 타입이란

### 1. 자료형으로서의 타입

- 데이터 타입(자료형)
    - 여러 종류의 데이터를 식별하는 분류 체계로 컴파일러에 값의 형태를 알려줌
- 자바스크립트의 데이터 타입(자료형)
    - undefined
    - null
    - Boolean
    - String
    - Symbol
    - Numeric(Num,ber와 BigInt)
    - Object

### 2. 집합으로서의 타입

- 타입은 값이 가질 수 있는 유효한 범위의 집합
- 타입을 제한하면 타입스크립트 컴파일러는 함수를 호출할 때 호환되는 인자로 호출했는지를 판단
- 올바르지 않은 타입의 값으로 함수를 호출하면 타입스크립트 컴파일러는 에러를 발생시킴

### 3. 정적 타입과 동적 타입

- 타입을 결정하는 시점에 따라 정적 타입과 동적 타입으로 분류됨
- 정적 타입 시스템
    - 모든 변수의 타입이 컴파일타임에 결정
    - 코드 수준에서 개발자가 타입을 명시해줘야함
    e.g. C, 자바, 타입스크립트
    - 컴파일타임에 타입 에러를 발견할 수 있음 → 프로그램의 안정성 보장
- 동적 타입 시스템
    - 변수의 타입이 런타임에서 결정
    - 개발자가 직접 타입을 정의해줄 필요 ❌
    e.g. 자바스크립트, 파이썬
    - 프로그램 실행시 타입 에러가 발견됨 → 프로그램에 오류가 생길 가능성 존재

### 4. 강타입과 약타입

- 암묵적 타입 변환
    - 개발자가 타입을 명시하거나 바꾸지 않았는데도 컴파일러 또는 엔진 등에 의해 런타임에 타입이 자동으로 변경되는 것
- 강타입
    - 암묵적 타입 변환 ❌
    - 서로 다른 타입을 갖는 값끼리 연산을 시도했을 때 컴파일러 또는 인터프리터에서 에러가 발생
    - 파이썬, 루비, 타입스크립트
- 약타입
    - 암묵적 타입 변환 ⭕
    - 서로 다른 타입을 갖는 값끼리 연산을 시도했을 때 컴파일러 또는 인터프리터가 내부적으로 판단 후 특정 값의 타입을 변환하여 연산을 수행 후 값을 도출
    - 자바스크립트, 자바, C++
- 타입 시스템
    - 타입 검사기가 프로그램에 타입을 할당하는데 사용하는 규칙 집합
    - 타입 시스템 종류
    - 
        - 컴파일러에 명시적으로 알려줘야하는 타입 시스템
        - 자동으로 타입을 추론하는 타입 시스템
        - 참고로 타입스크립트는 두 가지 타입 시스템의 영향을 받아서 두 방식 중 개발자가 선택할 수 있음.

### 5. 컴파일 방식

- 컴파일
    - 일반적인 컴파일
        - 사람이 이해할 수 있는 방식으로 작성한 코드(자바 등의 고수준 언어로 작성한 소스코드) → 컴퓨터가 이해할 수 있는 기계어(바이너리코드)로 변환
    - 타입스크립트의 컴파일
        - 타입스크립트 코드에서 타입이 모두 제거된 자바스크립트 코드로 변환

## 2.2 타입스크립트의 타입 시스템

### 1. 타입 애너테이션 방식

- 변수나 상수, 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 컴파일러에게 어떤 타입의 값이 저장될 것인지 직접 알려주는 문법
- 타입스크립트는 변수 뒤에 **`: type`** 구문을 붙여 데이터 타입을 명시
- 타입스크립트는 자바스크립트 코드에 점진적으로 타입을 적용 할 수 있음
→  **`: type` 을 제거해도 코드가 정상적으로 동작**

### 2. 구조적 타이핑

- 타입을 사용하는 프로그래밍 언어에서 값이나 객체는 하나의 구체적인 타입을 가짐
- 타입은 이름으로 구분되며 컴파일타임 이후에도 남아있음 
→ 이걸 명목적으로 구체화한 타입 시스템이라고 부름
- 타입스크립트에서 타입을 구분하는 방식
    - **구조적 타이핑** → 이름으로 타입을 구분하지 않고 **구조로 타입을 구분함**

### 3. 구조적 서브 타이핑

- 타입스크립트의 타입은 값의 집합
- 타입은 집합에 포함되는 값, 특정 값은 많은 집합에 포함될 수 있음
    
    **→ 타입스크립트에서는 특정 값이 string 또는 number 타입을 동시에 가질 수 있음**
    
    ```tsx
    type stringOrNumber = string | number;
    ```
    
- 구조적 서브 타이핑
    - 위처럼 집합으로 나타낼 수 있는 타입스크립트의 타입시스템의 기반 개념
    - 객체가 가지고 있는 속성(프로퍼티)를 바탕으로 타입을 구분
    - 타입스크립트는 이름이 다른 객체라도 **가진 속성이 동일**하다면 서로 호환이 가능한 **동일한 타입**으로 여김
    
    ```tsx
    interface Pet {
    	name: string;
    }
    let cat = { name: "Zag", age:2 };
    function greet(pet: Pet){
    	console.log("Hello, " + pet.name);
    }
    greet(cat); // Hello, Zag
    ```
    
    - greet 함수의 인자는 Pet 타입으로 제한되어있음
    - 타입을 명시하지 않은 cat 객체를 greet() 함수의 인자로 전달해도 코드는 정상적으로 실행됨
        - cat 객체는 Pet 인터페이스가 가지고 있는 name 속성을 가지고 있어서 pet.name 방식으로 name 속성에 접근이 가능함
        - 이런 타이핑 방식이 `구조적 타이핑`
- 타입스크립트의 서브타이핑(타입의 상속) 또한 구조적 타이핑을 기반으로 함
    
    ```tsx
    class Person {
    	name: string;
    	age: number;
    	
    	constructor(name: string, age: number){
    		this.name = name;
    		this.age = age;
    	}
    }
    
    class Developer {
    	name: string;
    	age: number;
    	sleepTime: number;
    	
    	constructor(name: string, age: number, sleepTime: number){
    		this.name = name;
    		this.age = age;
    		this.sleepTime = sleepTime;
    	}
    }
    
    function greet(p: Person){
    	console.log(`Hello, I'm ${p.name}`);
    }
    const developer = new Developer("zig", 20, 7);
    greet(developer); // Hello, I'm zig
    ```
    
- Developer 클래스가 Person 클래스를 상속받지 않았지만 greet(developer) 는 정상적으로 동작
→ Developer는 Person이 갖고 있는 속성을 가지기 때문
- **서로 다른 두 타입간의 호환성은 타입 내부의 구조에 의해 결정됨**
    
    → 타입이 계층 구조로 부터 자유로움
    

### 4. 자바스크립트를 닮은 타입스크립트

- 구조적 서브 타이핑 ↔ 명목적 타이핑
- 명목적 타이핑
    - 타입의 구조가 아닌 타입의 이름만으로 구별
    - 구조적으로 동일해도 이름이 다르면 다른 타입으로 취급
    - 타입의 동일성을 확인하는 과정에서 구조적 타이핑에 비해 조금 더 안전
- 타입스크립트가 구조적 타이핑을 채택한 이유
    - 덕 타이핑을 기반으로 하는 자바스크립트를 모델링한 언어이기 때문
    - 🦆 덕 타이핑(duck typing)
        - 어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식
- 덕 타이핑과 구조적 타이핑
    - 객체 변수, 메서드 같은 필드를 기반으로 타입을 검사한다는 점에서 동일
    - 타입을 검사하는 시점에서 차이를 가짐
        - 덕 타이핑 : 런타임에 타입 검사(동적 타이핑에서 사용)
        - 구조적 타이핑 : 컴파일타임에 타입 체커가 타입 검사(정적 타이핑에서 사용)

### 5. 구조적 타이핑의 결과

- 타입스크립트 구조적 타이핑의 특징 때문에 예기치 못한 결과가 나올 수 있음
    
    ```tsx
    interface Cube {
    	width: number;
    	height: number;
    	depth: number;
    }
    
    function addLines(c: Cube){
    	let total = 0;
    	for(const axis of Object.keys(c)){
    		const length = c[axis];
    	// c[axis] 에서 에러 발생
    	// Element implicitly has an 'any' type because expression of type 'string' 
    	// can't be used to index type 'Cube'.
      // No index signature with a parameter of type 'string' was found on type 'Cube'.
    		total += length;
    	}
    }
    ```
    
- 타입스크립트는 c[axis]가 어떤 속성을 지닐지 알 수 없고, c[axis] 타입을 number로 확정할 수 없어서 에러를 발생시킴
    
    → 타입스크립트 구조적 타이핑의 특징 때문에 Cube 타입 값이 들어갈 곳에 `name:string` 같은 속성이 추가된 객체도 할당이 가능하기 때문
    
- 이런 한계를 극복하고자 타입스크립트에 명목적 타이핑 언어의 특징을 가미한 **`식별할 수 있는 유니온(Discriminated Unions)`** 같은 방법이 생겨남

### 6. 타입스크립트의 점진적 타입 확인

- 점진적 타입 검사
    - 컴파일 타입에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식
    - 타입을 지정한 변수와 표현식은 정적으로 타입을 검사
    - 타입 선언이 생략되면 동적으로 검사를 수행(암시적 타입 변환이 일어남)
- 이런 특징 때문에 타입스크립트의 타입 시스템은 정적 타입의 정확성을 100% 보장하지 않음
    
    → 모든 타입을 컴파일러타임에 검사하지 않아도되기 때문에 타입이 올바르게 정해지지 않으면 런타임에서 에러가 발생하기도함
    
- `any 타입`
    - 타입스크립트 내 모든 타입의 종류를 포함하는 가장 상위 타입
    - 어떤 타입 값이든 할당 가능
    - 타입스크립트 컴파일 옵션인 `noImplicitAny` 값이 true 일 땐 에러가 발생

### 7. 자바스크립트 슈퍼셋으로서의 타입스크립트

- 타입스크립트는 자바스크립트의 상위 집합
- 타입스크립트 문법은 모든 자바스크립트의 문법을 포함

### 8. 값 vs 타입

- 값(value)
    - 프로그램이 처리하기 위해 메모리에 저장하는 모든 데이터
    - 프로그래밍 관점에서 문자열, 숫자, 변수, 매개변수 등이 값에 해당
    - 객체 또한 값임
    - 자바스크립트에서는 함수도 값임
        
        → 자바스크립트 함수는 런타임에 객체로 변환되기 때문
        
- 타입(type)
    - 타입스크립트는 변수, 매개변수, 객체 속성 등에 `: type` 형태로 타입을 명시함
    - `type`이나 `interface` 키워드로 커스텀 타입 정의 가능
- **값 공간과 타입 공간의 이름은 서로 충돌하지 않음**
→ 타입과 변수를 같은 이름으로 정의 가능!
⇒ 타입스크립트 문법인 type으로 선언한 내용은 자바스크립트 런타임에서 제거되기 때문에 충돌 ❌
- 타입스크립트에서 값과 타입의 구분은 맥락에 따라 달라지지 때문에 값 공간과 타입 공간을 혼동할 때도 있음
    
    ```tsx
    // 1번
    function email(options:{person: Person; subject:string; body:string}){
    	// ...
    }
    // 2번
    // 위 코드를 자바스크립트의 구조분해할당을 사용하면 아래와 같음
    function email({person, subject, body}){
    	// ...
    }
    // 3번
    // 위 코드를 타입스크립트 코드로 변경하면 아래와 같은데 에러가 발생함
    function email({person: Person, string, string}){
    	// ... 
    }
    ```
    
    - 값의 관점에서 Person과 string이 해석되었기 때문에 오류가 발생
    - 개발자의 의도는 매개 변수 객체의 속성인 person을 Person 타입으로, subject와 body는 string 타입으로 타입을 제한한 것
    - 하지만 3번 코드에서 Person, string이 값 공간에 있는 것으로 해석되고, person과 Person은 각 함수의 매개변수 객체 내부 속성의 키-값 쌍에 해당하는 것으로 해석됨
- 이처럼 값-타입 공간을 혼동하는 문제를 해결하기 위해 **값과 타입을 구분해서 작성**해야함
    
    ```tsx
    function email({
    	person,
    	subject,
    	body,
    }: {
    	person: Person;
    	subject: string;
    	body: string;
    }){
    	// ...
    }
    ```
    
- 타입스크립트에는 값과 타입 공간에 **동시에 존재**하는 심볼도 있음
    - **클래스와 enum**
- 클래스
    - 객체 인스턴스를 더욱 쉽게 생성하기 위한 문법기능(syntactic sugar) 실제 동작은 함수와 같음
    - 클래스는 타입으로도 사용됨
        
        → 클래스는 값과 타입 공간 모두에 포함 가능
        
    
    ```tsx
    class Developer {
    	name: string;
    	domain: string;
    	constructor(name: string, domain: string){
    		this.name = name;
    		this.domain = domain;
    	}
    }
    
    const me: Developer = new Developer("zig", "frontend");
    ```
    
    - 변수명 me 뒤의 : Developer에서 Developer는 타입에 해당
    - new 키워드 뒤의 Developer는 클래스의 생성자 함수인 값으로 동작
        
        → **타입스크립트에서 클래스는 타입 애너테이션으로 사용할 수 있지만 런타임에서 객체로 변환되어 자바스크립트의 값으로 사용됨**
        
- enum
    - 클래스와 마찬가지로 런타임에 객체로 변환되는 값
    - 런타임에 실제 객체로 존재하며 함수로 표현 가능
    - 클래스처럼 타입 공간에서 타입을 제한하는 역할을 하지만 자바스크립트 런타임에서 실제 값으로 사용될 수 있음
    - enum이 타입으로 사용되는 예시
        
        ```tsx
        enum WeekDays {
        	MON = "Mon",
        	TUES = "Tues",
        	WEDNES = "Wednes",
        	THURS = "Thurs",
        	FRI = "Fri",
        }
        // 'MON' | 'TUES' | 'WEDNES' | 'THURS' | 'FRI'
        type WeekDaysKey = keyof typeof WeekDays;
        function printDay(key: WeekDaysKey, message: string){
        	const day = WeekDays[key];
        	if(day <= WeekDays.WEDNES){
        		console.log(`It's still ${day}day, ${message}`);
        	}
        }
        printDay("TUES", "wanna go home");
        ```
        
        - WeekDays enum에 keyof typeof 연산자를 사용해서 type WeekDaysKey를 만들어서 printDay() 함수의 key 인자에 넘겨줄 수 있는 값의 타입을 제한
    - enum 이 값 공간에서 사용되는 예시
        
        ```tsx
        enum MyColors {
        	BLUE = "#0000FF",
        	YELLOW = "#FFFF00",
        	MINT = "#2AC1BC",
        }
        function whatMintColor(palette:{MINT: string}){
        	return palette.MINT;
        }
        
        console.log(whatMintColor(MyColors)); //  "#2AC1BC" 
        ```
        
        - 여기서 MyColors enum은 일반적인 객체처럼 동작
        - whatMintColor() 함수의 인자인 palette는 MINT라는 속성을 갖는 객체고 MyColors는 string 타입의 MINT 속성을 가지고 있기 때문에 위 코드가 정상적으로 실행됨
- 타입스크립트에서 자바스크립트의 키워드가 해석되는 방식
    
    
    | 키워드 | 값 | 타입 |
    | --- | --- | --- |
    | class | ⭕ | ⭕ |
    | const, let, var | ⭕ | ❌ |
    | enum | ⭕ | ⭕ |
    | function | ⭕ | ❌ |
    | interface | ❌ | ⭕ |
    | type | ❌ | ⭕ |
    | namespace | ⭕ | ❌ |

### 9. 타입을 확인하는 방법

- 타입스크립트에서 typeof, instanceof, 타입 단언을 사용해서 타입 확인 가능
- typeof
    - 연산하기 전에 피연산자의 데이터 타입을 나타내는 문자열을 반환
    - 반환 값
        - 자바스크립트의 7가지 기본 데이터 타입(Boolean, null, undefined, Number, BigInt, String, Symbol)과 Function(함수), 호스트객체, object객체
    - typeof 연산자는 값에서 쓰일 때와 타입에서 쓰일 때의 역할이 다름
        - 값에서 사용된 typeof는 자바스크립트 런타임의 typeof 연산자가 됨
        - 타입에서 사용된 typeof는 값을 읽고 타입스크립트 타입을 반환함
            
            ```tsx
            interface Person {
            	first: string;
            	last: string;
            }
            const person: Person = {first:"zig", last:"song"};
            
            const v1 = typeof person; // 값은 'object'
            type T1 = typeof person; // 타입은 Person
            // person 변수가 interface Person 타입으로 선언되어있기 때문에
            // 타입 공간에서의 typeof person은 Person을 반환
            ```
            
    - 자바스크립트의 클래스는 typeof 연산자를 쓸 때 주의해야함
- instanceof
    - 프로토타입 체이닝 어딘가에 생성자의 프로토타입 속성이 존재하는지 판단 가능
    - typeof 연산자처럼 필터링으로 타입이 보장된 상태에서 안전하게 값의 타입을 정제하여 사용 가능
- 타입 단언
    - as 키워드를 사용
    - 개발자가 해당 값의 타입을 더 잘 파악할 수 있을 때 사용되며 강제 형 변환과 유사한 기능 제공
    - 컴파일 단계에서는 타입 단언이 형 변환을 강제할 수 있지만 런타임에서는 효과 없음
        
        → 타입스크립트 코드는 자바스크립트로 변환되고 타입스크립트의 타입 시스템과 문법은 컴파일 단계에서 제거되기 때문
        
- 타입 가드
    - 특정 조건을 검사해서 타입을 정제하고 타입 안정성을 높이는 패턴

## 2.3 원시 타입

- 자바스크립트의 7가지 원시 값은 타입스크립트에서 원시 타입으로 존재함
- 원시 값과 원시 래퍼 객체
    - 자바스크립트의 내장 타입은 파스칼 표기법으로 표기함
    - 타입스크립트에서는 이에 대응하는 타입을 소문자로 표기함
    - 타입을 파스칼 표기법으로 표기하면 자바스크립트에서 원시 래퍼 객체라고 부름
    - 원시래퍼 객체는 원시 값이 아닌 객체 → 타입스크립트에서는 내장 원시 타입에 해당하는 타입을 파스칼 표기법으로 쓰지 않도록 주의해야함
- 파스칼 표기법
    - 단어의 첫 글자를 대문자로 쓰고 나머지 글자는 소문자로 쓰는 방식

### 1. boolean

- true와 false 값만 할당 가능
- 자바스크립트에는 boolean 원시 값은 아니지만 형 변환을 통해 true, false로 취급되는 Truthy, Falsy 값이 존재함
    
    → 이 값은 boolean 원시 값이 아니므로 타입스크립트에서도 boolean 타입에 비해당
    

### 2. undefined

- 정의되지 않았다는 의미의 타입
- 일반적으로 초기화되지 않은 값이나 존재하지 않음을 나타냄

### 3. null

- null 만 할당 가능
- 자바스크립트에서는 보통 빈 값을 할당해야할 때 null을 사용
    
    → 명시적, 의도적으로 값이 아직 비어있음을 보여줌
    
- 자바스크립트에서 값이 없다는 것을 나타낼 때 undefined와 null을 혼용해서 사용하지만 둘은 따로 존재하는 원시 값이기 때문에 서로의 타입에 할당할 수 없음

### 4. number

- 자바스크립트의 숫자에 해당하는 모든 원시 값 할당 가능
- 자바스크립트의 숫자는 정수, 부동소수점수를 구분하지않음 → 모두 number 타입에 할당 가능
- 자바스크립트에서 숫자에 해당하는 원시값 중 NaN이나 Infinity도 포함됨

### 5. bigInt

- ES2020에서 새롭게 도입된 데이터 타입(타입스크립트 3.2 버전부터 사용 가능)
- 이전 자바스크립트에서 가장 큰 수인 Number.MAX_SAFE_INTEGER를 넘어가는 값을 처리할 수 있음
- number 타입 ≠ bigInt 타입

### 6. string

- 문자열을 할당할 수 있는 타입
- 공백도 string 타입에 해당됨

### 7. symbol

- ES2015에서 도입된 데이터 타입
- Symbol() 함수를 사용하면 어떤 값과도 중복되지 않는 유일한 값 생성 가능
- 타입스크립트에는 symbol 타입과 const 선언에서만 사용할 수 있는 `unique symbol` 타입이라는 symbol의 하위 타입이 존재
- null과 undefined는 tsconfig 옵션이나 사용자의 취향에 따라 다르게 사용될 수 있음
- 타입스크립트의 모든 타입은 기본적으로 null과 undefined를 포함
    
    → ts-config의 strictNullCheks 옵션을 활성화하면 사용자가 명시적으로 해당 타입에 null이나 undefined를 포함시켜야 사용가능함
    

## 2.4 객체 타입

- 위에서 언급한 7가지 원시 타입에 속하지 않는 값은 모두 객체 타입
- 타입스크립트에서는 다양한 형태를 가지는 객체마다 개별적으로 타입 지정 가능
    
    → 배열 또는 클래스를 타입으로 지정 할 수 있고 복잡한 구조를 가진 객체도 타입으로 만들 수 있음
    

### 1. object

- 자바스크립트 객체의 정의에 맞게 대응되는 타입스크립트 타입 시스템은 object 타입
- object 타입은 가급적 사용하지 말도록 권장됨
    
    → any 타입과 유사하게 객체에 해당하는 모든 타입 값을 유동적으로 할당할 수 있어 정적 타이핑 의미가 퇴색되기 때문
    
- any 타입과 달리 원시 타입에 해당하는 값은 object 타입에 속하지 않음
    
    ```tsx
    function isObject(value: object){
    	return (
    		Object.prototype.toString.call(value).replace(/\[|\]|\s|object/g, "") === "Object"
    	);
    }
    // 객체, 배열, 정규표현식, 함수, 클래스 등 모두 object 타입과 호환됨
    isObject({});
    isObject({name:"gyeol"});
    isObject([0,1,2]);
    isObject(new RegExp("object"));
    isObject(function(){
    	console.log("hello");
    });
    
    // 원시타입은 호환 불가
    isObject(20); // false
    isObject("gyeol"); // false
    ```
    

### 2. {}

- 중괄호는 자바스크립트에서 객체 리터럴 방식으로 객체를 생성할 때 사용
- 타입스크립트에서 객체를 타이핑할 때도 중괄호를 사용할 수 있음
    
    → 중괄호 안에 객체의 속성 타입을 지정해주는 식으로 사용
    
    ⇒ **타이핑되는 객체가 중괄호 안에서 선언된 구조와 일치해야한다는 것을 의미**
    
    ```tsx
    // 정상
    const noticePopup: {title:string; description: string} = {
    	title: "공지사항",
    	description: "공지사항 테스트",
    };
    // SyntaxError
    const noticePopup: {title:string; description: string} = {
    	title: "공지사항",
    	description: "공지사항 테스트",
    	startAt: "2024.09.22 20:30:00", // startAt은 지정한 타입에 존재하지 않음 -> 오류
    };
    ```

    ![image](https://github.com/user-attachments/assets/88239a3d-ed89-4640-80d4-ba554a8490c9)
    
- 자바스크립트에서 빈 객체를 생성하기 위해 const obj = {}; 와 같은 구문 사용 가능
- 타입스크립트 또한 이에 대응하는 타입으로 {}를 사용 가능
→ {} 타입으로 지정된 객체에는 어떤 값도 속성으로 할당이 불가능
- 빈 객체 타입을 지정하는 바람직한 방법
    - 위처럼 {}를 사용하는 것보다 유틸리티 타입으로 Record<string, never> 처럼 사용하는 것이 바람직함
- {} 타입으로 지정된 객체는 완전히 비어있는 순수한 객체를 의미하지 않음
    - 자바스크립트 프로토타입 체이닝으로 Object 객체 래퍼에서 제공하는 속성에는 정상적으로 접근할 수 있음
    
    ```tsx
    let noticePopup: {} = {};
    noticePopup.title = "공지사항"; // Property 'title' does not exist on type '{}'
    console.log(noticePopup.toString()); // [object Object]
    ```
    

### 3. array

- 자바스크립트에서는 흔히 사용하는 객체 자료구조 외 배열, 함수, 정규식 등이 객체 범주에 속함
- 타입스크립트에서는 각 객체에 타입 지정 가능
- 타입스크립트 배열 타입(array)
    - 배열을 array라는 타입으로 다룸
    - 하나의 타입 값만 가질 수 있다는 점에서 자바스크립트 배열보다 엄격
    - 자바스크립트와 마찬가지로 원소 개수는 타입에 영향을 주지 않음
- 타입스크립트에서 배열 타입을 선언하는 방식
    - Array 키워드로 선언
    - 대괄호([])를 사용해서 선언
        
        → 두 방식은 결과적으로 같으므로 취향이나 컨벤션에 따라 선택할 것
        
- 주의
    - 튜플 타입도 대괄호로 선언함
    - 타입스크립트 튜플 타입은 배열과 유사하지만,
    튜플의 대괄호 내부에는 선언 시점에 지정해준 타입 값만 할당이 가능.
    원소 개수도 타입 선언 시점에 미리 정해짐
    → 객체 리터럴에서 선언하지 않은 속성을 할당하거나 선언한 속성을 할당하지 않을 때 에러가 발생한다는 점과 비슷
        
        ```tsx
        const targetCodes : ["NAME", "AGE"] = ["NAME", "AGE"]; // 정상 동작
        const targetCodes : ["NAME", "AGE"] = ["NAME", "AGE", "GENDER"];
        // 위 코드는 에러 발생
        //Type '["NAME", "AGE", string]' is not assignable to type '["NAME", "AGE"]'.
        //  Source has 3 element(s) but target allows only 2.
        ```
        

### 4. type과 interface 키워드

- 타입스크립트 object 타입은 실무에서 잘 사용하지 않음
- 보통 객체를 타이핑 하기 위해서는 타입스크립트에서만 독자적으로 사용할 수 있는 키워드를 사용
- 보통 객체를 타이핑 하기 위해 `type`과 `interface` 키워드를 자주 사용함
- 중괄호를 사용한 객체 리터럴 방식의 타입 지정은 중복적인 요소가 많음
- type과 interface 키워드를 사용하면 중복 없이 해당 타입을 사용할 수 있음
    
    ```tsx
    type NoticePopupType ={
    	title:string; 
    	description: string;
    };
    
    interface INoticePopup {
    	title:string; 
    	description: string;	
    };
    
    const noticePopup1: NoticePopupType = { ... };
    const noticePopup2: INoticePopup = { ... };
    
    ```
    
    - 타입스크립트에서는 변수 타입을 명시적으로 선언하지 않아도 컴파일러가 타입을 추론함
        
        → 타입스크립트 컴파일러가 변수 사용 방식과 할당된 값의 타입을 분석해서 타입을 유추하므로 모든 변수에 타입을 일일이 명시적으로 선언할 필요가 없음
        
    
    ✔ 컴파일러에 타입 추론을 온전히 맡길 것인지 명시적으로 타입을 선언할 것인지는 취향이나 컨벤션에 따라 다를 수 있음
    

### 5. function

- 자바스크립트에서 함수는 일종의 객체로 간주하지만 **function이라는 별도 타입으로 분류함**
- 타입스크립트에서도 함수를 별도 함수 타입으로 지정할 수 있음
- 주의
    - 자바스크립트에서 typeof 연산자로 확인한 function이라는 키워드 자체를 타입으로 사용하지 않음
    - 타입스크립트에서는 함수의 매개변수도 별도 타입으로 지정해야함
    - 함수가 반환하는 값이 있다면 반환값의 타입도 지정해야함
    
    ```tsx
    function add(a: number, b: number):number {
    	return a + b;
    }
    ```
    
- 함수 자체의 타입을 지정하는 방법
    - 호출 시그니처를 정의하는 방식 사용
    - 호출 시그니처(Call Signature)
        - 타입스크립트에서 함수 타입을 정의할 때 사용하는 문법
        - 함수 타입은 해당 함수가 받는 매개변수와 반환 값의 타입으로 결정됨
        - 호출시그니처는 이러한 함수의 매개변수와 반환값의 타입을 명시하는 역할
        
        ```tsx
        type add = (a:number, b:number) => number;
        ```
        
        - 호출 시그니처는 자바스크립트의 화살표 함수와 맥락이 유사함
        - 타입스크립트에서 함수 자체의 타입을 명시할 때는 화살표 함수 방식으로만 호출 시그니처를 정의함

# 📌 질문

1. **type과 interface의 차이에 대해 설명해주세요.**
2. **아래 코드와 같은 배열 변수 할당에서 `readonly` 에 대한 옳은 설명을 고르세요.**
    
    ```tsx
    const codeNames: readonly string[] = ['Coding', 'God'];
    ```
    
    1. 읽기만 가능하고 수정할 수 없다.
    2. 요소 추가만 가능하고 배열의 요소 삭제는 불가능하다.
    3. private으로 만들어져서 생성된 파일 내에서만 사용 가능하다.
    4. 클린 코드를 위해 읽기만 가능하다.

3. **타입스크립트에서 튜플과 배열은 타입 관점에서 동일한 개념이다.
(TRUE/FALSE로 답변 + 이유도 함께 설명해주세요.)**
4. **아래 타입스크립트 코드를 보고 a,b,c 의 타입에 대해 설명해주세요.**

	```tsx
	const a = "Hello World"
	let b = "Hello World"
	const c: string = "Hello World"
	```
