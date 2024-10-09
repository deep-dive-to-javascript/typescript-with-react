# 3장 고급 타입

2024.09.29 dasom



## 3.1 타입스크립트 만의 독자적 타입 시스템

> 타입스크립트의 타입 시스템은 자바스크립트에서 기인한 것이긴 하지만, 자바스크립트에서 제시되지 않은 독자적 타입 시스템을 가지고 있다. 
>
> 예를 들어 typeof any를 통해 any 타입을 살펴보면, 콘솔에서 "any"타입이 출력되지 않는다.



### 1. any 타입

any타입은 자바스크립트에 존재하는 모든 값을 오류없이 받을 수 있다. (== 자바스크립트에서 타입을 명시하지 않는 것과 동일한 효과)

따라서 any타입을 변수에 할당하는 것은 지양해야한다.

그럼에도 불구하고 any타입을 어쩔 수 없이 사용하는 대표적 경우

* 개발 단계에서 임시로 값을 지정할 때
* 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
* 값을 예측할 수 없을 때



### 2. unknown 타입

> unknown타입은 타입스크립트만의 타입 시스템이다. (자바스크립트에서 대응되는 자료형을 찾을 수 없다.)
>
> 하지만, any타입과의 유사점이 존재한다.

any타입과의 공통점

	* 어떤 타입이든 해당 타입(any || unknown)에 할당 가능 

any타입과의 차이점

* any타입은 어떤 타입으로도 할당 가능 (단, never는 제외)
* unknown타입은 any 타입 외에 다른 타입으로 할당 불가능

```js
let unknownValue: unknown;

unknownValue = 100; // 어떤 타입이든 할당가능
let someVal1: any = unknownValue; // any타입으로 선언된 변수는 할당 가능
let someVal2: number = unknownValue; // (X) 그 외 다른 변수는 할당 불가
```



그렇다면 unknown타입이 추가된 이유는?

> unknown 타입은 할당 시점에서는 에러가 발생하지 않지만, 실행하면 에러가 발생한다. 
>
> 즉, 개발자로하여름 타입 검사를 강제하는 의도를 가지고 있다.
>
> (따라서 예상할 수 없는 데이터 타입에 대해서는 unknown을 쓸 것을 권고한다.)



### 3. void 타입

> 함수가 어떤 값을 반환하지 않는 경우에는 void를 지정하여 사용한다.



> 🤖 chatGPT와의 대화
>
> Q. interface를 만들 때, 함수타입을 Function으로 지정해도 돼?
>
> A. 함수 타입을 정의할 때 `Function`을 사용하는 것보다는 함수의 시그니처를 명시하는 것이 권장됩니다. 구체적으로, 반환 값이 없는 함수라면 `void` 반환 타입을 명시하는 것이 더 명확하고 안전한 방식입니다.



자바스크립트에서는 함수에서 명시적 반환문을 작성하지 않으면 undefined가 반환된다. 하지만 타입스크립트에서는 void 타입이 사용된다. (!== undefined)

void타입은 변수에도 할당할 수 있긴 하지만, 대부분 무의미하다. (undefined아니면 null값만 할당 가능)



### 4. never 타입 

> never 타입은 값을 반환할수 "없는" 타입이다.

값을 반환할 수 없는 경우

* 에러를 던지는 경우
* 무한히 함수가 실행되는 경우



never타입은 모든 타입의 하위타입으로, 자신을 제외한 어떠한 타입도 never에 할당될 수 없다. (심지어 any타입도)



### 5. Array 타입

자바스크립트에서도 Object.prototype.toString.call(...) 연산자를 통해 Array 키워드를 확인할 수 있다. 하지만 엄밀히 말하면 자바스크립트에서는 배열을 객체에 속하는 타입으로 분류한다는 점과 타입스크립트는 Array를 위해 특수한 문법을 써야한다는 점에서 해당 절에서 다루게 되었다.

cf1. 배열을 Array가 아닌 []로 명시하는 경우 타입은 배열보다 더 좁은 범위인 튜플을 가리킨다. 

cf2. 자바스크립트에서는 배열에 숫자, 문자, 함수 등 다양한 값을 삽입할 수 있다.



배열타입의 선언

(2가지 방식은 선언하는 형식 외 차이점이 없으므로, 컨벤션에 따라 설정하면 된다)

* number[]과 같이 타입[] 방식
  * 여러 타입을 관리한다면 number[] | string[] 혹은 (number | string)]
* Array<number>와 같이 Array<타입> 방식
  * 여러 타입을 관리한다면 Array<number|string>



튜플의 사용

* 대괄호 안에 타입 시스템을 기술하여 사용한다.
* 괄호 안에 선언하는 타입의 개수가 튜플이 가질 수 있는 원소의 개수이다. 즉 특정 인덱스에 정해진 타입을 선언한다.
* let tuple: [number, string, boolean] = [1, 'string', true];
* 튜플은 배열과 달리 배열의 길이까지 제한하여 원소의 개수와 타입을 보장한다.
* 예시
  * react의 useState를 보면 const [value, setValue] = useState(false);와 같이 생겼는데, 배열 안에 첫번째 원소와 두번째 원소의 타입과 의미가 명확하다. 
* 옵셔널 프로퍼티의 명시
  * const optionalTuple1: [number, number, number?] = [1, 2] // 3번째 인덱스의 원소는 있어도 되고 없어도 됨



### 6. enum 타입

> enum 타입은 열거형이라고도 불리는, 타입스크립트에서 지원하는 특수한 타입이다.
>
> enum은 일종의 구조체를 만드는 타입으로, 열거형을 정의할 수 있다.



enum의 특징

* 각 멤버에 대해 명시적으로 값을 할당할 수 잇으며, 일부 멤버에 할당하지 않은 경우 타입스크립트는 누락된 멤버에 값을 1씩 늘려가며 자동으로 할당한다

```js
enum ProgrammingLanguage {
  Typescript = "Typescript",
  Javascript = "Javascript",
  Java = 300,
  Python = 400,
  Kotlin, // 401,
  Go, // 402
}
```

* enum 타입은 주로 문자열 상수를 생성하는데 사용된다
* 열거형을 타입으로 가지는 변수는 해당 열거형이 가지는 모든 멤버를 값으로 받을 수 있다

```js
enum ItemStatusType {
  DELIVERY_READY = "DELIVERY_READY",
  DELIVERING = "DELIVERING"
}

const checkItemAvailable = (itemStatus: ItemStatusType) => {
  switch (itemStatus){
    case ItemStatusType.DELIVERY_READY:
    case ItemStatusType.DELIVERING:
      return false;
	}
}
```



enum 사용시 주의점

* 숫자로만 이루어져있거단 타입스크립트가 자동으로 추론한 열거형은 안전하지 않다
* 단, const enum으로 열거형을 선언하면, 역방향으로의 접근은 허용하지 않는다
* 그럼에도 숫자 상수로 관리되는 열거형은 선언한 값 외의 값을 할당하거나 접근할 때 이를 방지할 수 없다
* 열거형은 타입 공간과 값 공간에서 모두 사용된다. 따라서 일부 번들러의 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 문제가 발생할 수 있다. (이 문제는 const enum 또는 as const assertion으로 해결할 수 있다.)





## 3.2 타입 조합

### 1. 교차 차입

여러가지 타입을 결합하여 하나의 탄일 타입으로 만들 수 있다

cf. 인터페이스는 extends를 사용해 교차타입을 만든다.

```js
type AType = {
  id: number
}

type BType = {
  name: string
}

type ABType = A & B
```



### 2. 유니온 타입

타입 중 하나가 될 수 있는 타입을 말한다.

cf. 인터페이스는 유니온 타입을 만들 수 없다. type을 사용해야 한다.

```js
type AType = {
  id: number
}

type BType = {
  name: string
}

type AorBType = A | B
```



### 3. 인덱스 시그니처

특정 타입의 속성 이름은 알 수 없지만 속성값의 타입을 알고있을 때 사용한다.

```js
interface MixedType {
  [key: string]: string | number; // 문자열 인덱스 시그니처
  age: number;                    // 고정된 속성
  is_adult: boolean;     
  // 에러 발생!! -> 인덱스 시그니처의 키가 string일 때는 string, number 타입만 오게되어있음. number면 된다.
}
```



### 4. 인덱스드 엑세스 타입

인덱스드 엑세스 타입은 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용된다.

```js
type User = {
  id: number;
  username: string;
  isAdmin: boolean;
};

type IdOrUsername = User['id' | 'username'];  // number | string 타입

```



### 5. 맵드 타입

다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법이다. 인덱스드 시그니처 문법을 사용해 반복적 타입 선언을 줄일 수 있다.

맵드 타입의 활용 예시 (from 🤖 chatGPT )

* 모든 속성을 `optional`로 만드는 맵드 타입

```typescript
type Optional<T> = {
  [K in keyof T]?: T[K];
};

interface Person {
  name: string;
  age: number;
}

type OptionalPerson = Optional<Person>;
// OptionalPerson 타입은 모든 속성이 선택적으로 변환됨
// { name?: string; age?: number; }
```

* 모든 속성을 `readonly`로 만드는 맵드 타입

```typescript
type Readonly<T> = {
  [K in keyof T]: Readonly<T[K]>;
};

type ReadonlyPerson = Readonly<Person>;
// { readonly name: string; readonly age: number; }
```

* 모든 속성의 타입을 `nullable`로 만드는 맵드 타입

```typescript
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type NullablePerson = Nullable<Person>;
// { name: string | null; age: number | null; }
```

* 모든 속성의 타입을 특정 타입으로 변환

```typescript
type AllStrings<T> = {
  [K in keyof T]: string;
};

type StringPerson = AllStrings<Person>;
// { name: string; age: string; }
```

* 조건부 맵드 타입

```typescript
type NullableFields<T> = {
  [K in keyof T]: T[K] extends string ? T[K] | null : T[K];
};

type NullableStringPerson = NullableFields<Person>;
// { name: string | null; age: number; }
```

위 예시에서 `T[K] extends string ? T[K] | null : T[K]`는 `T[K]`가 문자열 타입이면 `null`을 추가하고, 그렇지 않으면 원래 타입을 그대로 유지한다.

* as 키워드를 활용한 맵드 타입

```typescript
type RemapKeys<T> = {
  [K in keyof T as `new_${K & string}`]: T[K];
};

type RemappedPerson = RemapKeys<Person>;
// { new_name: string; new_age: number; }
```

이 예시는 기존 키에 `new_`를 붙여 새로운 객체 타입을 생성한다.



### 6. 템플릿 리터럴 타입

자바스크립트의 템플릿 시터럴 문자열을 사용해 문자열 리터럴 타입을 선언할 수 있는 문법이다.

```ts
type Stage = 
| "init"
| "select-image"
| "edit-image"

type StageName = `${Stage}-stage`;
// 'init-stage' | 'select-image-stage' | 'edit-image-stage'
```



### 7. 제네릭

정적 언어에서 다양한 타입 간 재사용성을 높이기 위해 사용하는 문법이다.

> 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워둔 다음에, 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 올 타입을 지정하여 사용하는 방식을 말한다.

타입 변수는 일반적으로 <T>, <E>, <K>, <V> 등을 사용한다. (함수를 호출할 때 반드시 <> 안에 타입을 명시해야하는 것은 아니다.)

any와는 엄연히 다른 타입으로, 예를 들어 제네릭을 사용하면 배열 요소가 동일한 타입이라고 보장할 수 있다.

유의점

* 제네릭은 일반화된 데이터 타입이기 때문에 배열에만 존재하는 length 속성을 제네릭에서 참조하려고 하거나 하면 에러가 발생한다.
* 파일 확장자가 tsx일 때 함수에 제네릭을 사용하면 에러가 발생한다. (태그의 꺾쇠괄호와 혼동되어 문제가 생김)
  * 이 상황을 피하기 위해서는 제네릭 부분에 extends 키워드를 사용한다.
  * e.g. const arrowEx = <T extends {}>(arg: T): T[] => {}



## 3.3 제네릭 사용법



### 1. 함수의 제네릭

```js
function ReadOnlyRepo<T>(target: ObjectType<T> | EntitySchema<T> | string): Repository<T>{
  return getConnection("ro").getRepository(target);
}

// cf. getConnection("ro") -> read-only로 DB 연결
```



### 2. 호출 시그니처의 제네릭

호출 시그니처는 타입스크립틔 함수 타입 문법으로, 함수의 매개변수와 반환 타입을 미리 선언하는 것을 말한다.

제네릭을 호출하는 호출 시그니처의 기본 예시

```ts
type IdentityFunction = <T>(arg: T) => T;
```



### 3. 제네릭 클래스

제네릭 클래스는 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스이다.

```ts
class GenericClass<T> {
  value: T;

  constructor(value: T) {
    this.value = value;
  }

  getValue(): T {
    return this.value;
  }
}
```



### 4. 제한된 제네릭 

타입 매개변수에 대한 제약 조건을 설정하는 기능이다

예를 들어 string 타입으로 제약하려면 타입 매개변수는 특정 타입을 상속 해야 한다

```ts
type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never 
? Partial<Record<Key,boolean>>
: naver;
```



### 5. 확정된 제네릭

타입 매개변수에 유니온 타입을 상속해서 선언하면 유연성을 잃지 않으면서 제약할 수 있다

<Key extends string | number>



### 6. 제네릭 예시

실제 가장 많이 활용되는 케이스는 API 응답 값의 타입을 지정할 때이다

```ts
export interface MobileApiResponse<Data>{
  data: Data;
  statusCode: string;
}

// 활용
export const fetchPriceInfo = ():Promise<MobileApiResponse<PriceInfo>> => {
  const priceUrl = "~"
  
  return request({
    method: "GET",
    url: priceUrl
  })
}
```



제네릭 사용시 유의할 점

* 제네릭을 굳이 사용하지 않아도 되는 타입에서는 사용하지 않기
* any 사용 지양하기
* 가독성 고려하기



## 질문

Q. 배열과 튜플의 차이는 무엇인가요?

