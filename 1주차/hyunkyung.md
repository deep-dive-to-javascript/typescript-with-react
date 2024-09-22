## 1장 들어가며
### 1.1 웹 개발의 역사
1. 자바스크립트의 탄생
- 1990년대에는 인터넷 익스플로러와 넷스케이프 내비게이터가 가장 많이 사용되는 웹 브라우저였음
- 1995년 브랜든 아이크가 웹의 다양한 콘텐츠를 표현하기 위해 자바스크립트 개발
- C, Java와 유사한 문법을 가지고 있으며 객체 지향 언어인 셀프의 프로토타입 기반 상속 개념과 Lisp 계열 언어 중 하나인 스킴의 일급 함수 개념을 차용
  - 함수를 값처럼 다룰 수 있음(변수에 할당되거나, 인수로 전달되거나, 반환될 수 있음)
- 완벽한 해결책보다 빠르게 시장에 내놓는 것을 목표로 개발 10일만에 완성..

2. 자바스크립트 표준 ECMAScript의 탄생
- 넷스케이프와 MS의 브라우저 전쟁속에서 크로스 브라우징 이슈가 발생했음
- 또한 자바스크립트와 브라우저 발전 속도 간의 차이가 나기 시작했고 브라우저는 자바스크립트의 변화를 따라가지 못했음
- 새로운 버전의 브라우저가 자바스크립트의 새로운 기능을 지원하더라도 사용자가 예전 버전을 사용한다면 새로운 기능을 사용할 수 없었음
    - 이를 해결하기 위해 폴리필과 트랜스파일 같은 개념 등장
      - 폴리필: 브라우저가 지원하지 않는 코드를 브라우저에서 사용할 수 있도록 변환한 코드 조각이나 플러그인
        ```javascript
        if (!Array.prototype.includes) {
        Array.prototype.includes = function(element) {
             return this.indexOf(element) !== -1;
           };
        }
        ```
      - 트랜스파일: 최신 버전의 코드를 예전 버전의 코드로 변환하는 과정
- 따라서 jQuery 같이 브라우저 호환성을 고민하지 않고 한번에 개발할 수 있도록 지원해주는 라이브러리가 유행
- 이러한 상황에서 자바스크립트 표준화가 필요하다는 목소리가 커지게 되었고, 이를 위해 ECMA 인터내셔널이라는 기구에서 자바스크립트 표준화를 진행하게 됨

- 관련자료
  - [드림코딩 앨리: 자바스크립트의 역사](https://youtu.be/wcsVjmHrUQg?si=8F0EjjiavCqS4jGm) -> [요약한 블로그](https://dev-ellachoi.tistory.com/m/102)

3. 웹사이트에서 웹 애플리케이션으로의 전환

**웹사이트**
- 단방향으로 정보를 제공하는 정적인 페이지

**웹 애플리케이션**
- 사용자와 상호작용이 가능한 동적인 페이지

4. 개발 생태계의 발전
- 거대한 웹 애플리케이션이 등장하면서 하나의 웹 페이지를 통으로 개발하는 것이 아니라, 컴포넌트 단위로 개발하는 방식 생겨남
- 또한 Ajax로 페이지 전체를 새로고침하지 않아도 자바스크립트의 비동기 요청을 사용해서 페이지의 일부 데이터를 로드할 수 있게 되었음
- 이러한 변화로 다뤄야하는 데이터가 폭발적으로 늘어나고 표현해야 하는 화면도, 디바이스도 다양해지면서 웹 개발의 복잡성이 증대

**CBD(Component Based Development) 방법론 등장**
- 서비스에서 다루는 데이터를 구분하고 그에 맞는 UI를 표현할 수 있게 컴포넌트 단위로 개발하는 접근 방식
- 컴포넌트 단위로 개발하면서 재사용성이 높아지고 유지보수가 쉬워짐

5. 개발자 협업의 필요성 증가
- 개발에 투입된 인원이 많아지고 개발하는 서비스의 규모가 커지면서 개발자 간의 협업이 필수적이었고 자바스크립트의 한계가 드러남

### 1.2 자바스크립트의 한계
1. 동적 타입 언어
- 코드가 실행되는 런타임에 변수 타입이 결정
2. 동적 타이핑 시스템의 한계
```javascript
const sumNumber = (a, b) => {
  return a + b;
};

sumNumber(1, 2); // 3
```
이 코드를 실행하면 어떤 에러도 발생하지 않고 정상적으로 동작

```javascript
const sumNumber = (a, b) => {
  return a + b;
};

sumNumber(100); // NaN
sumNumber("a", "b") // "ab"
```
- 인자에 하나의 숫자만 전달하거나 숫자가 아닌 문자를 전달했지만 어떤 에러도 발생하지 않고 정상적으로 동작
- 개발자의 의도와는 다르게 동작할 수 있게 되는 것

3. 한계 극복을 위한 해결 방안
- JSDoc
  - API 문서 생성도구로 타입 및 에러 확인이 가능하고 타입힌트를 제공하는 HTML 문서를 생성
  - 어디까지나 주석의 성격을 지니기 때문에 강제성을 부여해 사용하기 어려움
- propTypes
  - 리액트에서 사용하는 라이브러리로 컴포넌트에 전달되는 props의 타입을 검사
  - 전체 애플리케이션의 타입 검사를 하는 데는 사용할 수 없고 리액트에서만 사용 가능
- 다트
  - 구글이 자바스크립트를 대체하기 위해 제시한 새로운 언어로 타이핑이 가능
  - 이미 자바스크립트가 자리매김한 상황에 사용자들이 새로운 언어를 배워야 하는 부담이 있음 

4. 타입스크립트의 등장
- 시간이 지나 MS는 자바스크립트의 슈퍼셋 언어인 타입스크립트를 공개
    - 슈퍼셋: 기존 언어에 새로운 기능과 문법을 추가해서 보완하거나 향상하는 것. 기존 언어와 호환되며 일반적으로 컴파일러 등으로 기존 언어 코드로 변환되어 실행됨
- 다트와 달리 자바스크립트 코드를 그대로 사용할 수 있었고 아래와 같은 단점을 극복할 수 있어 많은 환영을 받았음
  - 안정성 보장
    - 정적 타이핑을 제공하여 컴파일 단계에서 타입 검사를 해주기 때문에 런타임 에러 사전에 방지
  - 개발 생산성 향상
    - IDE에서 타입 자동 완성 기능을 제공하여 변수와 함수 타입 추론 가능
    - 리액트를 사용할 때 어떤 prop을 넘겨야 하는지 매번 확인하지 않아도 사용부에서 바로 볼 수 있기 때문에 개발 생산성 향상 
  - 협업에 유리
    - 인터페이스, 제네릭 등을 지원하여 코드를 더 쉽게 이해할 수 있게 도와줌
      - 타입스크립트 인터페이스: 객체 구조를 정의하는 역할을 하며 객체가 그 구조를 따르게 함
    - 협업 과정에서 자동완성 기능이나 기술된 인터페이스로 코드 파악이 쉽게 도와줌
  - 자바스크립트에 점진적으로 적용 가능
    - 자바스크립트의 슈퍼셋이기 때문에 일괄 전환이 아닌 점진적 도입이 가능

## 2장 타입
### 2.1 타입이란
1. 자료형으로서의 타입
- 변수란 값을 저장할 수 있는 공간이자 값을 가리키는 상징적인 이름
- 변수에 저장되는 값은 데이터 타입에 따라 다르게 처리됨
- 자바스크립트는 다음과 같은 7가지 데이터 타입(자료형)을 정의
  - undefined, null, Boolean, String, Symbol, Numeric, Object
- 데이터 타입은 여러 종류의 데이터를 식별하는 분류 체계로 컴파일러에 값의 형태를 알려줌
- 개발자는 타입을 사용해서 값의 종류를 명시할 수 있고 메모리를 더욱 효율적으로 사용할 수 있음

2. 집합으로서의 타입
- 프로그래밍에서의 타입은 수학의 집합과 유사
- 타입은 값이 가질 수 있는 유효한 범위의 집합을 의미
```javascript
const num: number = 123;
const str: string = "abc";

function func(n: number){
}

func(num);
func(str); // Argument of type 'string' is not assignable to parameter of type 'number'
```
- 어떤 값이 T 타입이라면 컴파일러(또는 개발자)는 이 값으로 어떤 일을 할 수 있고, 어떤 일을 할 수 없는지를 사전에 알 수 있음
- 타입 시스템은 코드에서 사용되는 유효한 값의 범위를 제한해서 런타임에서 발생할 수 있는 유효하지 않은 값에 대한 에러를 방지해줌
- 위의 예시에서는 func()라는 함수의 인자로 number 타입 값만 할당할 수 있도록 제한되어 있음.
- 마치 집합의 경계처럼 함수의 인자를 Number 타입의 집합으로 제한하는 것

3. 정적 타입과 동적 타입
- 타입을 결정하는 시점에 따라 타입을 정적 타입과 동적 타입으로 분류
- 정적 타입: 컴파일 시점에 타입이 결정되는 것
  - C, C++, Java, C# 등
  - 컴파일 타임에 타입을 검사하기 때문에 런타임에 타입 에러가 발생할 확률이 낮음
- 동적 타입: 런타임 시점에 타입이 결정되는 것
  - 자바스크립트, 파이썬 등
    - 왜 파이썬은 타입스크립트 같은 대안이 없을까?
      - 파이썬은 주로 데이터 분석, 인공지능, 머신러닝 등의 분야에서 사용되는데 이러한 분야에서는 동적 타입이 더 유리하기 때문
      - 데이터 분석에서는 데이터의 형태가 다양하고 데이터의 형태가 바뀔 수 있고 빠르게 수정하고 확인하는 과정이 반복되므로 동적 타입이 더 유리
  - 개발 과정에서 에러 없이 마음껏 코드를 작성할 수 있지만 런타임 시점에 타입 에러가 발생할 수 있음

4. 강타입과 약타입
- 암묵적 타입 변환: 개발자가 의도적으로 타입을 명시하거나 바꾸지 않았는데도 컴파일러 또는 엔진 등에 의해 런타임에 타입이 자동으로 변경되는 것
- 암묵적 타입 변환 여부에 따라 강타입과 약타입으로 분류
  - 강타입: 서로 다른 타입을 갖는 값끼리 연산을 시도하면 에러가 발생
  - 약타입: 서로 다른 타입을 갖는 값끼리 연산을 시도하면 자동으로 타입을 변환하여 연산을 수행
- 자바스크립트는 약타입 언어로서 타입 변환을 자동으로 수행
  - 편리함을 제공하지만 개발자가 의도하지 않은 결과를 초래할 수 있음
- 약타입 언어의 부작용을 방지하기 위해 타입을 명시해서 코드를 작성하고 개발자의 의도가 논리적으로 합당한지 검사하는 기준 필요
- 타입 검사기가 프로그램에 타입을 할당하는 데 사용하는 규칙 집합을 타입 시스템이라고 하며 크게 두가지로 구분함
  - 어떤 타입을 사용하는지를 컴파일러에 명시적으로 알려줘야 하는 타입 시스템
  - 자동으로 타입을 추론하는 타입 시스템
- 타입스크립트는 두 가지 타입 시스템의 영향을 모두 받았고 선택 가능

5. 컴파일 방식
- 컴파일의 일반적인 의미는 사람이 이해할 수 있는 방식으로 작성한 코드를 컴퓨터가 이해할 수 있는 기계어로 변환하는 과정을 의미
- 타입스크립트의 컴파일 결과물은 여전히 사람이 이해할 수 있는 방식인 자바스크립트 파일이라는 점에서 차이가 있음

### 2.2 타입스크립트의 타입 시스템
1. 타입 애너테이션 방식
- 타입 애너테이션: 변수나 상수 혹은 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법
- 타입스크립트는 변수 이름 뒤에 `: type` 구문을 붙여 데이터 타입을 명시
- 기존 자바스크립트 코드에 점진적으로 타입을 적용할 수 있는 특징을 가지고 있어 `: type` 선언부를 제거해도 코드가 정상적으로 동작

```typescript
  let isDone: boolean = false;
  let decimal: number = 6;
```

2. 구조적 타이핑
- 타입을 사용한 여러 프로그래밍 언어에서 값이나 객체는 하나의 구체적인 타입을 가지고 있음
- 타입은 이름으로 구분되며 컴파일타임 이후에도 남아있음 -> 명목적으로 구체화한 타입 시스템이라고 불림
```java
class Developer {
  public int faceValue;

  public Developer(int faceValue) {
  this.faceValue = faceValue;
}
}

class BankNote {
  public int faceValue;

  public BankNote(int faceValue) {
  this.faceValue = faceValue;
}
}

public class Main {
  public static void main(String[] args) {
  Developer developer = new Developer(1000);
  BankNote bankNote = new BankNote(1000);

  // developer = bankNote;  // 컴파일 오류 발생
  // bankNote = developer;  // 컴파일 오류 발생
}
```
- 또한 서로 다른 클래스끼리 명확한 상속 관계나 공통으로 가지고 있는 인터페이스가 없다면 타입은 서로 호환되지 않음
```java
interface Valuable {
    int getFaceValue();
}

class Developer implements Valuable {
    private int faceValue;

    public Developer(int faceValue) {
        this.faceValue = faceValue;
    }

    public int getFaceValue() {
        return faceValue;
    }
}

class BankNote implements Valuable {
    private int faceValue;

    public BankNote(int faceValue) {
        this.faceValue = faceValue;
    }

    public int getFaceValue() {
        return faceValue;
    }
}

public class Main {
    public static void main(String[] args) {
        Developer developer = new Developer(1000);
        BankNote bankNote = new BankNote(1000);

        Valuable valuableDeveloper = developer; // OK
        Valuable valuableBankNote = bankNote;   // OK
    }
}
```
- 그러나 타입스크립트에서 타입을 구분하는 방식은 조금 다름.
- 이름으로 타입을 구분하는 명목적인 타입 언어의 특징과 달리 타입스크립트는 구조로 타입을 구분 -> 구조적 타이핑
```typescript
interface Developer{
    faceValue: number;
}

interface BankNote {
    faceValue: number;
}

let developer: Developer = { faceValue: 1000 };
let bankNote: BankNote = { faceValue: 1000 };

developer = bankNote; // OK
bankNote = developer; // OK
```

3. 구조적 서브타이핑
- 구조적 서브타이핑
  - 객체가 가지고 있는 속성을 바탕으로 타입을 구분하는 것 
  - 이름이 다른 객체라도 가진 속성이 동일하다면 타입스크립트는 서로 호환이 가능한 동일한 타입으로 여김
```typescript
interface Pet {
    name: string;
}

interface Cat {
    name: string;
    age: number;
}

let pet: Pet;
let cat: Cat = { name: "Lucy", age: 5 };

pet = cat; // OK
```
- Cat과 Pet은 다른 타입으로 선언되었지만 Pet이 가지고 있는 name이라는 속성을 가지고 있음
- 따라서 Cat 타입으로 선언한 cat을 Pet 타입으로 선언한 pet에 할당할 수 있음

**참고 자료**
[구조적 서브타이핑이 사용되는 경우](https://toss.tech/article/typescript-type-compatibility)

4. 자바스크립트를 닮은 타입스크립트
- 타입스크립트의 타입 시스템은 구조적 서브 타이핑 사용 <-> 명목적 타이핑
- 명목적 타이핑은 타입의 동일성을 확인하는 과정에서 구조적 타이핑에 비해 조금 더 안전하다는 장점이 있음
  - 개발자가 의도한 타입이 아니라면 변수에 타입을 명시하는 과정에서 에러 내뱉음
- 타입스크립트가 구조적 타이핑을 채택한 이유는 타입스크립트가 자바스크립트를 모델링한 언어이기 때문
  - 자바스크립트는 덕타이핑을 기반으로 함 -> 어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식
- 구조적 타이핑 덕분에 타입스크립트는 더욱 유연한 타이핑이 가능해짐
- 덕타이핑과 구조적 타이핑의 차이 -> 타입을 검사하는 시점
  - 덕 타이핑: 런타임에 타입 검사
  - 구조적 타이핑: 컴파일타임에 타입체커가 타입을 검사

5. 구조적 타이핑의 결과
- 구조적 타이핑의 특징 때문에 예기치 못한 결과가 나올 때도 있음
```typescript
interface Cube {
    width: number;
    height: number;
    depth: number;
}

function addLines(c: Cube){
    let total = 0;
  
    for (const axis of Object.keys(c)) {
      total += c[axis];
    }
}
```

- addLines() 함수의 매개변수인 c는 Cube 타입으로 선언되었고, Cube 인터페이스의 모든 필드는 number 타입을 가지기 때문에 c[axis]는 당연히 Number 타입이라고 예측할 수 있음
- 그러나 c에 들어올 객체는 어떤 속성이든 가질 수 있기 떄문에 c[axis]의 타입이 string일 수도 있어 에러가 발생
```typescript
const namedCube = {
    width: 100,
    height: 100,
    depth: 100,
    name: "Cube"
};

addLines(namedCube);
}
```

- 이러한 한계를 극복하고자 타입스크립트에 명목적 타이핑 언어의 특징을 가미한 [식별할 수 있는 유니온](https://radlohead.gitbook.io/typescript-deep-dive/type-system/discriminated-unions) 같은 방법이 생겨남

6. 타입스크립트의 점진적 타입 확인
- 타입스크립트는 점진적으로 타입을 확인하는 언어 -> 컴파일타임에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식
- 타입을 지정한 변수와 표현식은 정적으로 타입을 검사하지만 타입 선언이 생략되면 동적으로 검사 수행
- 타입 선언을 생략하면 암시적 타입 변환 일어남
```typescript
function add(x,y){
    return x + y;
}

// 위 코드는 아래와 같이 암시적 타입 변환이 일어남
function add(x: any, y: any): any
```
- 이러한 점진적 타이핑의 특징 덕분에 자바스크립트 코드를 타입스크립트로 마이그레이션할 때 타입 선언을 생략하고 코드를 작성할 수 있음
- 하지만 이러한 특징 때문에 타입 시스템은 정적 타입의 정확성을 100% 보장하지 않으므로 타입 선언을 최대한 명시적으로 작성하는 것이 좋음

**any 타입**
- 타입스크립트 내 모든 타입의 종류를 포함하는 가장 상위 타입 -> 어떤 값이든 할당할 수 있음
- 단, 타입스크립트 컴파일 옵션인 `noImplicitAny`를 사용하면 any 타입을 사용할 수 없음

7. 자바스크립트 슈퍼셋으로서의 타입스크립트
- 타입스크립트는 자바스크립트의 슈퍼셋 언어로서 자바스크립트의 모든 기능을 포함하고 있음
- 또한 타입스크립트 컴파일러는 타입스크립트뿐만 아니라 일반 자바스크립트 프로그램에서도 유용하게 사용할 수 있음
```javascript
let developer = "Colin";
console.log(developer.toUpperCase());

//이 코드를 타입스크립트 컴파일러로 실행하면 다음과 같은 오류 발생
// property 'toUpperCase' does not exist on type 'string'
// Did you mean 'toUpperCase'?

// 같은 코드를 자바스크립트 런타임에서만 실행하면 다음과 같은 에러를 던져줄뿐임
// developer.toUpperCase is not a function
```

8. 값 vs 타입
- 값은 프로그램이 처리하기 위해 메모리에 저장하는 모든 데이터
- 타입스크립트는 변수, 매개변수, 객체 속성 등에 :type 형태로 타입을 명시
- 값 공간과 타입 공간의 이름은 서로 충돌하지 않기 때문에 타입과 변수를 같은 이름으로 정의할 수 있음
  - type으로 선언한 내용은 자바스크립트 런타임에서 제거되기 때문에 값 공간과 타입 공간은 서로 충돌하지 않음
- 값과 타입은 타입스크립트에서 별도의 네임스페이스에 존재.
- 타입스크립트에서 값과 타입의 구분은 맥락에 따라 달라지기 때문에 값 공간과 타입 공간을 혼동할 때도 있음
```typescript
function email(options: { person: Person; subject: string; body: string }) {
    // ...
}
```
- 자바스크립트의 구조 분해 할당을 사용하면 아래와 같이 작성할 수 있지만
```javascript
function email({person, subject, body }) {
// ...
}
```

- 타입스크립트에서는 오류가 발생
```typescript
function email({ person: Person, subject: string, body: string }) {
// ...
}
```
- Person과 string이 타입이 아닌 값으로 해석되기 때문
- 따라서 아래와 같이 작성해야 함
```typescript
function email({ person, subject, body }: { person: Person; subject: string; body: string }) {}
```

- 타입스크립트에는 값과 타입 공간에 동시에 존재하는 심볼도 있음 

**클래스**
- 값인 동시에 타입으로도 사용되어 값과 타입 공간 모두에 포함될 수 있음

**enum**
```typescript
    enum Direction {
        Up,
        Down,
        Left,
        Right 
    }
```

- enum 예시를 순수 자바스크립트 코드로 컴파일하면 아래와 같음
```javascript
let Direction;
    (function (Direction) {
      Direction[(Direction.Up = 0)] = "Up";
      Direction[(Direction.Down = 1)] = "Down";
      Direction[(Direction.Left = 2)] = "Left";
      Direction[(Direction.Right = 3)] = "Right";
    })(Direction || (Direction = {}));
```

- enum이 타입으로 사용된 경우
```typescript
enum WeekDays {
  MON = "Mon",
  TUES = "Tues",
  WEDNES = "Wednes",
  THURS = "Thurs",
  FRI = "Fri",
}
// ‘MON’ | ‘TUES’ | ‘WEDNES’ | ‘THURS’ | ‘FRI’
type WeekDaysKey = keyof typeof WeekDays;

function printDay(key: WeekDaysKey, message: string) {
  const day = WeekDays[key];
  if (day <= WeekDays.WEDNES) {
    console.log(`It’s still ${day}day, ${message}`);
  }
}

printDay("TUES", "wanna go home");
```

- enum이 값 공간에서 사용된 경우
```typescript
enum MyColors {
  BLUE = "#0000FF",
  YELLOW = "#FFFF00",
  MINT = "#2AC1BC",
}

function whatMintColor(palette: { MINT: string }) {
  return palette.MINT;
}

whatMintColor(MyColors); // ✅
```

- 타입스크립트에서 자바스크립트의 키워드가 해석되는 방식

| 키워드 | 값 | 타입 |
| --- | --- | --- |
| class | ✅ | ✅ |
| enum | ✅ | ✅ |
| interface | ❌ | ✅ |
| type | ❌ | ✅ |
| namespace | ✅ | ❌ |
| function | ✅ | ❌ |
| const, let, var | ✅ | ❌ |

9. 타입을 확인하는 방법
- typeof, instanceof 그리고 타입 단언을 사용해서 타입을 확인할 수 있음

**typeof**
- typeof는 연산하기 전에 피연산자의 데이터 타입을 나타내는 문자열을 반환 - "number", "string", "boolean", "object", "function", "symbol", "undefined"
- typeof 연산자는 값에서 쓰일 때와 타입에서 쓰일 때의 역할이 다름

```typescript
interface Person {
    first: string;
    last: string;
}

const person: Person = { first: "Colin", last: "Firth" };

function email(options:{ person: Person, subject: string, body: string }) {
    // ...
}

//값에서 사용된 typeof는 자바스크립트 런타임의 typeof 연산자가 된다. 
const v1 = typeof person; // 값은 'object'
const v2 = typeof email; // 값은 'function'

//타입에서 사용된 typeof는 값을 읽고 타입스크립트 타입을 반환한다.
type T1 = typeof person; // 타입은 Person
type T2 = typeof email; // 타입은 (options: { person: Person; subject: string; body: string; }) => void
```

- 자바스크립트 클래스는 typeof 연산자를 쓸 때 주의해야 함. 
```typescript
class Developer {
    name: string;
    sleepingTime: number;
  
    constructor(name: string, sleepingTime: number) {
        this.name = name;
        this.sleepingTime = sleepingTime;
    }
}

const d = typeof Developer; // 값은 'function'
type T = typeof Developer; // 타입은 typeof Developer -> 생성자 함수의 타입

const zig: Developer = new Developer("Zig", 8);
type ZigType = typeof zig; // 타입은 Developer -> 인스턴스의 타입
```

- 자바스크립트의 클래스는 결국 함수이기 때문에 typeof Developer는 'function'을 반환
- typeof Developer를 풀어서 설명하면 아래와 같음
```
new (name: string, sleepingTime: number): Developer
```

**instanceof**
- 자바스크립트에서 instanceof 연산자를 사용하면 프로토타입 체이닝 어딘가에 생성자의 프로토타입 속성이 존재하는지 확인할 수 있음
- typeof 연산자처럼 instanceof 연산자의 필터링으로 타입이 보장된 상태에서 안전하게 값의 타입을 정제하여 사용 가능
```typescript
let error = unknown;

if (error instanceof Error) {
    console.error(error.message);
} else{
    throw Error("Unexpected error");
}
```

**타입 단언**
- 타입 단언이라 부르는 문법을 사용해서 타입을 강제할 수도 있는데 `as` 키워드를 사용
- 개발자가 해당 값의 타입을 더 잘 파악할 수 있을 때 사용되며 강제 형 변환과 유사한 기능을 제공

```typescript
const loaded_text: unknown; // 어딘가에서 Unknown 타입 값을 전달받았다고 가정

const validateInputText = (text: string) => {
    if(text.length < 10) return "최소 10글자 이상 입력해야 합니다";
    return "정상 입력된 값입니다"
}

validateInputText(loaded_text as string);
```

### 2.3 원시 타입

**원시 값과 원시 래퍼 객체**
- String, Number 등과 같은 원시 래퍼 객체가 존재하므로 이러한 원시값을 파스칼 표기법으로 쓰지 않도록 주의

1. boolean
- boolean은 true 또는 false 값을 가질 수 있는 타입
- 1이나 0 같은 truthy/falsy 값은 boolean 타입에 해당하지 않음

2. undefined
- 초기화되지 않았거나 존재하지 않음을 나타냄
```typescript
let value: string;
console.log(value); // undefined (값이 아직 할당되지 않았음)

type Person = {
    name: string;
    // optional로 지정되어있으므로 undefined 할당 가능
    age?: number;
}
```

3. null
- 명시적 & 의도적으로 값이 비어있을 수 있음을 보여줌
```typescript
// job이라는 속성이 있을 수도 없을수도 있음을 나타냄
type Person1 = {
    name: string;
    job?: string;
}

// job이라는 속성을 사람마다 갖고 있지만 값이 비어있을 수도 있다는 것을 나타냄
type Person2 = {
    name: string;
    job: string | null;
}
```

4. number
- 자바스크립트의 숫자에 해당하는 모든 원시 값을 할당할 수 있음
```typescript
const maxLength: number = 10;
const maxWidth: number = 120.3;
const maximum: number = Infinity;
const notANumber: number = NaN;
```

5. bigInt
- 이전의 자바스크립트에서는 가장 큰 수인 Number.MAX_SAFE_INTEGER를 넘어가는 수를 표현할 수 없었음
- ES2020에서 도입된 bigInt는 이러한 문제를 해결
```typescript
const bigNumber1: bigint = BigInt(999999999999);
const bigNumber2: bigint = 999999999999n;
```

6. string
```typescript
const name: string = "Colin";
const sentence: string = `My name is ${name}`;
```

7. symbol
- 어떤 값과도 중복되지 않는 유일한 값을 생성할 수 있음
```typescript
const MOVIE_TITLE = Symbol("title");
const MUSIC_TITLE = Symbol("title");

console.log(MOVIE_TITLE === MUSIC_TITLE); // false

let SYMBOL : unique symbol = Symbol(); // A variable whose type is a unique symbol must be 'const'
```

**참고자료**
[strictNullChecks](https://ohhako.github.io/kimhako/articles/2020-07/Angular-strictNullChecks-post-copy)

### 2.4 객체 타입
- 앞에서 언급한 7가지 원시 타입에 속하지 않는 값은 모두 객체 타입으로 분류할 수 있음
- 타입스크립트에서는 다양한 형태를 가지는 객체마다 개별적으로 타입을 지정할 수 있음
1. object
- 가급적 사용하지 말도록 권장됨 -> 모든 타입 값을 유동적으로 할당할 수 있어 정적 타이핑의 의미가 퇴색되기 떄문

```typescript
function isObject(value: object) {
  return (
          Object.prototype.toString.call(value).replace(/\[|\]|\s|object/g, "") ===
          "Object"
  );
}
// 객체, 배열, 정규 표현식, 함수, 클래스 등 모두 object 타입과 호환된다
isObject({});
isObject({ name: "KG" });
isObject([0, 1, 2]);
isObject(new RegExp("object"));
isObject(() => {
  console.log("hello wolrd");
});
isObject(class Class {});
// 그러나 원시 타입은 호환되지 않는다
isObject(20); // false
isObject("KG"); // false
```

2. {}
- 타입스크립트에서 객체를 타이핑할 때 중괄호 안에 객체의 속성 타입을 지정해주는 식으로 사용할 수 있음
- 타이핑되는 객체가 중괄호 안에서 선언된 구조와 일치해야 함
```typescript
// 정상
const noticePopup: { title: string; description: string } = {
  title: "IE 지원 종료 안내",
  description: "2022.07.15일부로 배민상회 IE 브라우저 지원을 종료합니다.",
};

// SyntaxError
const noticePopup: { title: string; description: string } = {
  title: "IE 지원 종료 안내",
  description: "2022.07.15일부로 배민상회 IE 브라우저 지원을 종료합니다.",
  startAt: "2022.07.15 10:00:00", // startAt은 지정한 타입에 존재하지 않으므로 오류
};
```

- 빈 객체를 생성하기 위해 {}를 사용할 수 있지만 어떤 값도 속성으로 할당할 수 없음
- {} 보다 Record<string, unknown>을 사용하는 것이 더 바람직
```typescript
let noticePopup: {} = {};

noticePopup.title = "IE 지원 종료 안내"; // (X) title 속성을 지정할 수 없음
```

3. array
- 타입스크립트 배열 타입은 하나의 타입 값만 가질 수 있다는 점에서 자바스크립트 배열보다 조금 더 엄격함
- 타입스크립트에서 배열 타입을 선언하는 방식은 Array 키워드로 선언하거나 []를 사용해서 선언하는 방법이 있음
```typescript
const getCartList = async (cartId: number[]) => {
  const res = await CartApi.GET_CART_LIST(cartId);
  return res.getData();
};

getCartList([]); // (O) 빈 배열도 가능하다
getCartList([1001]); // (O)
getCartList([1001, 1002, 1003]); // (O) number 타입 원소 몇 개가 들어와도 상관없다
getCartList([1001, "1002"]); // (X) ‘1002’는 string 타입이므로 불가하다
```

- 주의해야 할 점은 튜플 타입도 대괄호로 선언한다는 것
- 튜플의 대괄호 내부에는 선언 시점에 지정해준 타입 값만 할당할 수 있으며 원소 개수도 타입 선언 시점에 미리 정해짐
- 이것은 객체 리터럴에서 선언하지 않은 속성을 할당하거나, 선언한 속성을 할당하지 않았을 때 에러가 발생한다는 점과 비슷함
```typescript
const targetCodes: ["CATEGORY", "EXHIBITION"] = ["CATEGORY", "EXHIBITION"]; // (O)
const targetCodes: ["CATEGORY", "EXHIBITION"] = [
  "CATEGORY",
  "EXHIBITION",
  "SALE",
]; // (X) SALE은 지정할 수 없음
```

4. type과 interface 키워드
- 흔히 객체를 타이핑하기 위해 자주 사용하는 키워드로 type과 interface가 있음
```typescript
type NoticePopupType = {
  title: string;
  description: string;
};

interface INoticePopup {
  title: string;
  description: string;
}
const noticePopup1: NoticePopupType = { ... };
const noticePopup2: INoticePopup = { ... };
```

5. function
- 자바스크립트에서 함수도 일종의 객체로 간주하지만 function이라는 별도 타입으로 분류함
```javascript
function add(a, b) {
  return a + b;
}

console.log(typeof add); // ‘function’
```

- 마찬가지로 타입스크립트에서도 함수를 별도의 함수 타입으로 지정할 수 있음
- 함수 타입 지정시 주의할 점
  - 자바스크립트에서 typeof 연산자로 확인한 function 이라는 키워드 자체를 타입으로 사용하지 않음
  - 함수는 매개변수 목록을 받을 수 있는데 타입스크립트에서는 매개변수도 별도의 타입으로 지정해야 함

```javascript
function add(a: number, b: number): number {
  return a + b;
}
```

- 함수 자체의 타입을 지정할 때는 호출 시그니처를 정의하는 방식 사용
```typescript
// 자바스크립트의 화살표 함수와 맥락이 유사하다
type add = (a: number, b: number) => number;
```

### 질문
1. 정적 타입과 동적 타입의 차이와 장단점에 대해 설명해주세요.
2. 다음 코드에서 cat을 할당할 수 있는 이유는 무엇일까요? 
```typescript
interface Pet {
   name: string;
   }

interface Cat {
name: string;
age: number;
}

let pet: Pet;
let cat: Cat = { name: "Lucy", age: 5 };

pet = cat; // OK
```
