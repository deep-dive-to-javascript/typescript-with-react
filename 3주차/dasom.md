# 4장 타입 화장하기・좁히기

2024.10.09 dasom



## 4.1 타입 확장하기

> 타입스크립트에서는 extends, 교차 타입, 유니온 타입을 사용하여 타입을 확장하 ㄹ수 있다.



### 1. 타입 확장의 장점

타입 확장을 통해 코드 중복을 줄일 수 있다.

```ts
// interface -> extends로 확장
// 메뉴 요소 타입 
interface BaseMenuItem {
  itemName: string | null;
  intemImageUrl: string | null;
}
// 장바구니 요소 타입
interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}

// type -> &로 확장
type BaseMenuItem = {
  itemName: string | null;
  intemImageUrl: string | null;
}
type  BaseCartItem = {
  quantity: number;
} & BaseMenuItem
```



### 2. 유니온 타입

유니온 타입은 2개 이상의 타입을 조합하여 사용하는 방법이다. (합집합으로 해석할 수 있다.)

```ts
type MyUnion = A | B;
```

> 주의점
>
> MyUnion타입으로 선언된 값은 A와 B 모두가 가지고 있는 속성에만 접근할 수 있다.



### 3. 교차 타입

기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것이다.

```ts
type MyIntersection = A & B
```

> 유니온타입과 달리 A만 가지고 있는 속성이나 B만 가지고 있는 속성에도 접근할 수 있다. 

* 교차타입 사용 주의점

```ts
type A = string | number;
type B = number | boolean;
type C = A & B // number type만 교차가 가능하기 때문에 type C = number;가 된다.
// 단, 모든 타입이 호환되지 않을 경우는 never가 된다.
```



### 4. extends와 교차타입

extends 키워드를 사용한 타입이 교차 타입과 100% 상응하는 것은 아니다.

```ts
interface A {
  tip: number;
}
interface B extends A {
  tip: string;  // error!
}
// 호환불가 에러 발생한다

----------------------------

type C = {
  tip: number;
}
type D = C & {
  tip: string;  // error X
}
// 에러가 발생하지는 않지만, tip이라는 속성에 대해 호환되지 않는 타입이 선언되어 
// tip은 결국 never속성이 된다.
```



> ❗️Insight. 배민 메뉴 시스템에 타입 확장 적용하기
>
> 기존 타입에 무분별하게 속성을 추가하여 사용하는 것보다 타입을 확장해서 사용하는 것이 좋다. 라고 되어있다.
>
> 실제 코드를 작성할 때, A type에서 약간 변경된 B 타입이 추가되었던 적이 있었다.
>
> 이때 A타입과 B타입의 공통부분을 Base 타입으로 지정하고, 각각 A, B로 확장해 사용했더라면 매번 타입검사를 수행해야했던 어려움을 줄일 수 있지 않았을까 한다.





## 4.2 타입 좁히기 - 타입 가드

타입 좁히기를 통해 명시적 타입 추론과 타입 안정성 높이기가 가능하다.



### 1. 타입 가드에 따라 분기 처리하기

어떤 함수가 A | B 타입의 매개변수를 받을 때, A이냐 B에 따냐 로직을 처리하고 싶다고 가정하자.

if문을 사용할 수 없기 때문에 (컴파일하면 타입 정보는 사라져 런타임에 존재하지 않기 때문이다)

자바스크립트 typeof, instanceof, in과 같은 연산자를 사용해 런타임에 유효한 타입 가드를 만들 수 있다.



### 2. 원시 타입을 추론할 때: typeof 연산자 활용하기

자바스크립트 동작 박식으로 인해 null과 배열 타입 등이 object로 판별되는 등 복잡한 타입을 검증하는 데에는 한계가 존재한다. 따라서 원시 타입을 좁히는 용도로만 사용할 것을 권장한다.

typeof 연산자를 사용해 검사할 수 있는 타입 목록

* string
* number
* boolean
* undefined
* object
* function
* bigint
* symbol



### 3. 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

```ts
interface Range {
  start: Date;
  end: Date;
}

interface DatePickeProps {
  selectedDates?: Date | Range;
}

const DatePicker = ({selectedDates}: DatePickerProps) => {
  const [selected, setSelected] = useState(convertoToRange(selectedDates));
}

export function convertToRange(selecte? Date: Range): Range | undefined {
  return selected instanceof Date // 인스턴스화된 객체 타입을 판별
  ? {start: selected, end: selected}
  : slected;
}
```



### 4. 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

여러 객체 타입을 유니온 타입을 가지고 있을 때, in 연산자를 상요해 속성의 유무에 따라 조건 분기가 가능

```ts
interface BaseNoticeProps {
  title: string;
 	body: string;
}

interface NoticeWithCookieProps extends BaseNoticeProps {
  cookieKey: string;
}

export type NoticeProps = BaseNoticeProps | NoticeWithCookieProps

const Notice: React.FC<NoticeProps> = (props) => {
  if ("cookieKey" in props) return <NoticeWithCookie {...props} />;
  return <NoticeBase {...props} />;
}
```



### 5. is연산자로 사용자 정의 타입 가드 만들어 활용하기

```ts
// string 타입의 매개변수가 destinationCodeList 배열 원소 중 하나인지를 검사
const isDestinationCode = (x:string): x is DestinationCode => 
	destinationCodeList.includes(x);
```



> ❗️사용 예시 추가 공유
>
> 예약 상품에서 일반 예약과, 반복 예약 두가지 타입이 존재하는데 해당 예약에 따른 분기 처리가 필요했어요.
>
> 그래서 
>
> export const isRepeatReserveType = (thing: any): thing is RepeatReserveType => thing?.repeat_period !== undefined
>
> 위와 같은 타입 가드를 만들어 검사에 활용했습니다. 





## 4.3 타입 좁히기 - 식별할 수 있는 유니온 (Discriminated Union)

식별할 수 잇는 유니온 (= 태그된 유니온, Tagged Union)은 타입 좁히기에 사용되는 방식이다.

```ts
type ErrorFeedbackType = TextError | ToastError | AlertError;
```

위의 상황에서 ToastError만 가지는 필드와 AlertError만이 가지는 필드를 모두 가지는 객체가 있다면,

에러를 뱉어야할 것이지만 자바스크립트는 덕 타이핑 언어이기 때문에 별도의 타입에러를 뱉지 않는다.

이 때, 각 타입에

```ts
type TextError = {
  errorType: "TEXT";
  errorMessage: string;
}
type ToastError = {
  errorType: "TOAST";
  errorMessage: string;
  toastDuration: number;
}
type AlertError = {
  errorType: "ALERT";
  errorMessage: string;
  onConfirm: () => void;
}
```

errorType이라는 판별자를 달아주면 잘못 정의된 여러 객체에 대해 에러를 발생시킬 수 있다. 



식별할 수 있는 유니온 판별자로 사용할 수 있는 타입은 

* 리터럴 타입이어야 한다.
* 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스터늣와 할 수 있는 타입은 포함되지 않아야한다.

위의 조건을 만족해야 한다.

> cf. 유닛타입
>
> 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 나타내는 타입이다.
>
> null, undefined, 리터럴 타입을 포함해 true, 1 등 이 이에 해당한다.





### 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

 Exhaustiveness Checking이란 철저하게 타입을 검사하는 것을 말하며, 타입 좁히기에 사용되는 패러다임 중 하나다.



추가 예시 (from 🤖ChatGPT)

```ts
type Shape = 
  | { kind: 'circle', radius: number }
  | { kind: 'square', sideLength: number }
  | { kind: 'triangle', base: number, height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.sideLength ** 2;
    case 'triangle':
      return (shape.base * shape.height) / 2;
    default:
      // 모든 경우를 다루고 있는지 확인하기 위한 never 타입을 사용한 exhaustiveness checking
      const _exhaustiveCheck: never = shape;
      throw new Error(`Unhandled case: ${shape}`);
  }
}
```

* 새로운 케이스 추가 시

```ts
type Shape = 
  | { kind: 'circle', radius: number }
  | { kind: 'square', sideLength: number }
  | { kind: 'triangle', base: number, height: number }
  | { kind: 'rectangle', width: number, height: number }; // 새로운 타입 추가

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.sideLength ** 2;
    case 'triangle':
      return (shape.base * shape.height) / 2;
    default:
      const _exhaustiveCheck: never = shape; // TypeScript 오류 발생: 'rectangle' 타입이 처리되지 않음
      throw new Error(`Unhandled case: ${shape}`);
  }
}
```





## 질문

Q. extends와 교차 타입의 차이점을 설명해 주세요.

