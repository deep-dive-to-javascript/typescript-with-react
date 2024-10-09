## 4장 타입확장하기&좁히기
### 4.1 타입 확장하기
1. 타입 확장의 장점
- 타입 확장은 중복 제거, 명시적인 코드 작성 외에도 확장성이란 장점을 가지고 있음 ㅠㅎ

2. 유니온 타입
- 유니온 타입은 2개 이상의 타입을 조합하여 사용하는 방법
```typescript
// 합집합
// 집합 A의 모든 원소는 집합 MyUnion의 원소이며 집합 B의 모든 원소도 집합 MyUnion의 원소이다.
type MyUnion = A | B;
```
- 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있음

3. 교차 타입
- 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것
- 교집합의 개념과 비슷하다.
```typescript
// 교집합
// 집합 A의 모든 원소는 집합 MyIntersection의 원소이며 집합 B의 모든 원소도 집합 MyIntersection의 원소이다.
// MyIntersection의 모든 원소는 집합 A의 원소이자 집합 B의 원소이다
type MyIntersection = A & B;
```
- 교차 타입을 사용할 때 타입이 서로 호환되지 않는 경우도 있음
```typescript
type IdType = string | number;
type Numeric = number | boolean;

// Universal은 IdType과 Numeric의 교차 타입이므로 두 타입을 모두 만족하는 경우에만 유지됨
// 따라서 number 타입만 유지됨
type Universal = IdType & Numeric;
```

4. extends와 교차 타입 
extends 키워드를 사용해서 교차 타입을 만들 수도 있지만 100% 서로 호환되지 않아 주의해야 한다.
```typescript
interface DeliveryTip {
    tip: number;
}

interface Filter extends DeliveryTip {
    tip: string;
    // Interface 'Filter' incorrectly extends interface 'DeliveryTip'.
    // Types of property 'tip' are incompatible.
    // Type 'string' is not assignable to type 'number'
}

type FilterType = DeliveryTip & {
    tip: string;
}
```
interface extends를 사용해서 타입을 확장하면 같은 속성을 다른 타입으로 선언했을 때 위같은 에러가 발생한다.
하지만 type은 에러가 발생하지 않는다. 새롭게 추가되는 속성에 대해 미리 알수가 없기 때문이다.

5. 배달의 민족 메뉴 시스템에 타입 확장 적용하기
아래와 같은 기본 메뉴 타입이 있다고 가정하고 타입을 확장해보자
```typescript
type Menu = {
    name: string;
    image: string;
}
```
만약 눌러서 gif가 재생된다거나 이미지가 아닌 text를 보여주는 기능이 추가된다면 두가지 방법을 이용할 수 있다.
첫번째는 gif나 text를 optional하게 추가하는 것이다.
```typescript
type Menu = {
    name: string;
    image: string;
    gif?: string;
    text?: string;
}
```
이 방법은 text나 gif가 undefined일수도 있기 때문에 읽으려할때 type error가 발생할 수 있다.

두번째 방법은 Menu 타입을 확장하는 것이다.
```typescript
type Menu = {
    name: string;
    image: string;
}

type MenuWithGif = Menu & {
    gif: string;
}

type MenuWithText = Menu & {
    text: string;
}
```
이 방법은 타입이 명확해지기 때문에 에러가 발생하지 않고 의도도 확실하게 드러난다.

### 4.2 타입 좁히기 - 타입 가드
1. 타입가드에 따라 분기처리하기
- 타입 가드는 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능을 말함

2. 원시타입을 추론할 때 typeof 사용하기
- typeof 연산자를 사용하면 원시타입의 타입을 추론할 수 있다.
- 배열이나 객체는 모두 object로 추론되기 때문에 한계가 있다.
```typescript
const replaceHyphen: (Date | string) => Date | string = (date) => {
    if (typeof date === 'string') {
        return date.replace(/-/g, '/');
    }
    return date;
}
```

3. 인스턴스화된 객체 타입을 판별할 때 instanceof 사용하기
- instanceof 연산자를 사용하면 인스턴스화된 객체의 타입을 판별할 수 있다.
```typescript
interface Range {
    start: Date;
    end: Date;
}

export function convertToRange(selected?: Date | Range): Range | undefined {
    return selected instanceof Date ? {start: selected, end: selected} : selected;
}
```

4. 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기
- in 연산자를 사용하면 객체의 속성이 있는지 없는지에 따라 구분
- 프로토타입 체인으로 접근할 수 있는 속성이면 true를 반환
```typescript
interface BasicNoticeDialogProps {
  noticeTitle: string;
  noticeBody: string;
}

interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
  cookieKey: string;
  noForADay?: boolean;
  neverAgain?: boolean;
}

export type NoticeDialogProps = BasicNoticeDialogProps | NoticeDialogWithCookieProps;

const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if('cookieKey' in props) return <NoticeDialogWithCookie {...props} />;
  return <NoticeDialogBase {...props} />;
}
```

5. is 연산자로 사용자 정의 타입 가드 만들어 활용하기
- is 연산자를 사용하면 사용자 정의 타입 가드를 만들어 활용할 수 있다.
```typescript
const isDestinationCode = (x: string): x is DestinationCode => 
	destinationCodeList.includes(x);
```
위와 같이 사용자 정의 타입 가드를 만들어 사용하면 타입을 좁힐 수 있다.
만약 is를 사용하지않고 boolean 값만 리턴하게 만든다면 typescript는 x의 타입을 모르게 된다.

### 4.3 타입 좁히기 - 식별할 수 있는 유니온
- 자바스크립트는 덕타이핑 언어이기 때문에 유니온 타입을 작성했을 때 주의가 필요하다.
```typescript
type TextError = {
  errorCode: string;
  errorMessage: string;
}

type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}

type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void;
}

type ErrorFeedbackType = TextError | ToastError | AlertError;
```

위 같은 타입이 있을 때 아래와 같이 작성해도 에러를 내뱉지 않는다. 이렇게 되면 개발 과정에서 의미를 알 수 없는 에러를 발생시킬 수 있다.
```typescript
const errorArr: ErrorFeedbackType[] = [
  // ...
  {errorCode: '999', errorMessage: '잘못된 에러', onConfirm: () => {}, toastShowDuration: 3000},
];
```

2. 식별할 수 있는 유니온
위 같은 경우를 방지하기 위해 타입간의 구조 호환을 막기 위해한 식별할 수 있는 유니온이 필요하다.
```typescript
type TextError = {
  errorType: 'TEXT',
  errorCode: string;
  errorMessage: string;
}

type ToastError = {
  errorType: 'TOAST',
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}

type AlertError = {
  errorType: 'ALERT',
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void;
}

type ErrorFeedbackType = TextError | ToastError | AlertError;
```

3. 식별할 수 있는 유니온의 판별자 선정
- 식별할 수 있는 유니온 타입의 판별자는 유닛 타입(unit type)으로 선언되어야 정상적으로 동작
- 유닛 타입은 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값(=unique value)을 가지는 타입
- 리터럴 값과 같은 구체적이 값이 유닛 타입에 해당
- 공식 github 이슈 탭에 나온 정의
    - 리터럴 타입이어야 한다.
    - 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화 할 수 있는 타입(instantiable type)은 포함되지 않아야 한다.

```typescript
interface A {
    type: 'a';
    value: number;
}
```

### 4.4 Exhausiveness Checking으로 정확한 타입 분기 유지하기
- Exhaustiveness Checking이란 가능한 모든 타입 케이스에 대해 철저하게 타입을 검사하는 것을 말하며 타입 좁히기에 사용되는 패러다임 중 하나
```typescript
type ProductPrice = '10000' | '20000';

const getProductName = (productPrice: ProductPrice): string => {
  if(ProductPrice === '1000') return '배민상품권 1만원';
  if(ProductPrice === '2000') return '배민상품권 2만원';
  else {
    return '배민상품권';
  } 
}
```

위 코드는 각 상품권 가격에 따라 상품권 이름을 올바르게 반환하고 있어 문제가 없어보이지만, 이후에 새로운 상품권이 생기면서 문제가 발생할 가능성이 있다.

```typescript
type ProductPrice = '10000' | '20000' | '5000';

const getProductName = (productPrice: ProductPrice): string => {
  if(ProductPrice === '1000') return '배민상품권 1만원';
  if(ProductPrice === '2000') return '배민상품권 2만원';
  // if(ProductPrice === '5000') return '배민상품권 5천원';
  else {
    return '배민상품권';
  } 
}
```

ProductPrice 타입에 새롭게 '5000'을 추가했지만, getProductName 함수에는 분기처리를 추가되지 않았다. 이런 경우를 방지하기 위해 Exhaustiveness Checking을 사용한다.

```typescript
type ProductPrice = '10000' | '20000' | '5000';

const getProductName = (productPrice: ProductPrice): string => {
  if(ProductPrice === '1000') return '배민상품권 1만원';
  if(ProductPrice === '2000') return '배민상품권 2만원';
  // if(ProductPrice === '5000') return '배민상품권 5천원';
  else {
    exhaustiveCheck(productPrice);
    return '배민상품권';
  } 
}

const exhaustiveCheck  = (param: never) => {
  throw new Error('type error.');
}
```

exhaustiveCheck 함수는 매개변수를 never 타입으로 선언하고 있다. 즉 매개변수로 그 어떤 값도 받을 수 없으며, 만일 값이 들어온다면 에러를 내뱉는다. 
이 함수를 타입 처리 조건문의 마지막 else문에 사용하면 앞의 조건문에서 모든 타입에 대한 분기 처리를 강제할 수 있다.