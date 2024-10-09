## 4.1 타입 확장하기

> 타입스트크립트에서는 interface와 type 키워드를 사용해서 타입을 정의하고,
> extends, 교차 타입, 유니온 타입을 사용하여 타입을 확장한다.

### 타입 확장의 장점

- 코드 중복 제거
- 명시적인 코드 작성
- 확장성

#### interface를 이용한 타입 확장

```tsx
interface BaseMenuItem {
  itemName: string | null
  itemImageUrl: string | null
  itemDiscountAmount: number
  stock: number | null
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number
}
```

#### type을 이용한 타입 확장

```tsx
type BaseMenuItem = {
  itemName: string | null
  itemImageUrl: string | null
  itemDiscountAmount: number
  stock: number | null
}

type BaseCartItem = {
  quantity: number
} & BaseMenuItem
```

> [!NOTE]
>
> 타입스크립트의 타입은 속성의 집합이 아니라 **값의 집합**

### 유니온 타입

- 2개 이상의 타입을 조합하여 사용하는 방법
- 집합 관점으로 보면 **합집합**의 개념

```tsx
type Union = A | B
```

> 집합 A의 모든 원소는 집합 Union의 원소이며, 집합 B의 모든 원소 역시 집합 Union의 원소다.
> 즉, A 타입과 B 타입의 모든 값이 Union 타입의 값이 된다.

- 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근 가능

```tsx
interface CookingStep {
  orderId: string
  price: number
}

interface DeliveryStep {
  orderId: string
  time: number
  distance: string
}

function getDeliveryDistance(step: CookingStep | DeliveryStep) {
  return step.distance
  // Property 'distance' does not exist on type 'CookingStep | DeliveryStep'
  // Property 'distance' does not exist on type 'CookingStep'
}
```

> 즉, step이라는 유니온 타입은 `CookingStep` 또는 `DeliveryStep` 타입에 해당할 뿐이지 `CookingStep`이면서 `DeliveryStep`인 것은 아니다

### 교차 타입

- 기존 타입을 합쳐 **필요한 모든 기능을 가진** 하나의 타입으로 만드는 것
- 집합 관점으로 보면 **교집합**의 개념

```tsx
type Intersection = A & B
```

> 집합 Intersection의 원소는 집합 A의 원소이자 집합 B의 원소이다.
> 즉, Intersection 타입의 모든 값은 A 타입의 값이면서 B 타입의 값이다.

- 교차 타입으로 선언된 값은 교차 타입에 포함된 모든 타입을 갖고 있어야 함

```tsx
type BaedalProgress = CookingStep & DeliveryStep

function logBaedalInfo(progress: BaedalProgress) {
  console.log(`주문 금액: ${progress.price}`)
  console.log(`배달 거리: ${progress.distance}`)
}
```

> 따라서 BaedalProgress 타입의 progress 값은 CookingStep이 갖고 있는 price 속성과 DeliveryStep이 갖고 있는 distance 속성을 모두 포함하고 있다.

#### 교차 타입을 사용할 때 타입이 서로 호환되지 않는 경우

```tsx
type IdType = string | number
type Numeric = number | boolean

type Universal = IdType & Numeric // ?
```

### extends와 교차 타입

#### extends 키워드를 사용한 교차 타입 (확장)

```tsx
interface BaseMenuItem {
  itemName: string | null
  itemImageUrl: string | null
  itemDiscountAmount: number
  stock: number | null
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number
}
```

> BaseCartItem은 BaseMenuItem을 확장함으로써 BaseMenuItem의 속성을 모두 포함하고 있다.

- BaseCartItem: 상위 집합
- BaseMenuItem: 부분 집합
  >

#### type 키워드를 사용한 교차 타입

```tsx
type BaseMenuItem = {
  itemName: string | null
  itemImageUrl: string | null
  itemDiscountAmount: number
  stock: number | null
}

type BaseCartItem = {
  quantity: number
} & BaseMenuItem
```

#### extends 키워드 사용 시 주의 사항

- extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않을 수 있음

```tsx
interface DeliveryTip {
  tip: number
}

interface Filter extends DeliveryTip {
  tip: string
  // Interface 'Filter' incorrectly extends interface 'DeliveryTip'.
  // Types of property 'tip' are incompatible.
  // Type 'string' is not assignable to type 'number'.
}
```

```tsx
type DeliveryTip = {
  tip: number
}

type Filter = DeliveryTip & {
  tip: string // never
}
```

> type 키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문에 선언 시 에러가 발생하지 않는다. 하지만 tip이라는 같은 속성에 대해 서로 호환되지 않는 타입이 선언되어 결국 never 타입이 된다.

### 배달의 민족 메뉴 시스템에 타입 확장 적용하기

```tsx
interface Menu {
  name: string
  image: string
}

function MainMenu() {
  const menuList: Menu[] = [{ name: '찜', image: '찜.png' }]

  return (
    <ul>
      {menuList.map(menu => (
        <li>
          <img src={menu.image} />
          <span>{menu.name}</span>
        </li>
      ))}
    </ul>
  )
}
```

> #### 특정 메뉴의 중요도를 다르게 주기 위한 추가 요구사항
>
> 1. 특정 메뉴를 길게 누르면 git 파일이 재생되어야 한다.
> 2. 특정 메뉴는 이미지 대신 별도의 텍스트만 노출되어야 한다.

```tsx
const menuList = [
  { name: '찜', image: '찜.png' },
  { name: '찌개', image: '찌개.png' },
  { name: '회', image: '회.png' },
]

const specialMenuList = [
  { name: '돈까스', image: '돈까스.png', gif: '돈까스.gif' },
  { name: '피자', image: '피자.png', gif: '피자.gif' },
]

const packageMenuList = [
  { name: '1인분', image: '1인분.png', text: '1인 가구 맞춤형' },
  { name: '족발', image: '족발.png', text: '오늘은 족발로 결정' },
]
```

#### 하나의 타입에 여러 속성을 추가하기

- `Menu` 타입 하나로 각 메뉴 목록의 타입 정의 가능

```tsx
interface Menu {
    name: string
    image: string
    gif?: string
    text?: string
}

const menuList: Menu[] = [...]
const specialMenuList: Menu[] = [...]
const packageMenuList: Menu[] = [...]
```

```tsx
specialMenuList.map(menu => menu.text)
// TypeError: Cannot read properties of undefined
```

> specialMenuList도 Menu 타입의 원소를 갖기 때문에 text 속성에 접근이 가능하다. 하지만 specialMenuList 배열의 모든 원소는 text 속성을 갖고 있지 않으므로 컴파일 단계에서는 문제가 없지만 런타임에서 에러가 발생한다.

#### 타입을 확장하는 방식

- 각 배열의 타입을 확장할 타입에 맞게 명확히 규정
  - `Menu` / `SpecialMenu` / `PackageMenu`

```tsx
interface SpecialMenu extends Menu {
	gif: string
}

interface PackageMenu extends Menu {
	text: string
}

const menuList: Menu[] = [...]
const specialMenuList: SpecialMenu[] = [...]
const packageMenuList: PackageMenu[] = [...]
```

```tsx
specialMenuList.map(menu => menu.text)
// Property 'text' does not exist on type 'SpecialMenu'
```

> [!NOTE]
>
> #### 타입 확장의 이점
>
> - 적절한 네이밍을 사용해서 타입의 의도를 명확히 표현 가능
> - 예기치 못한 버그 예방

## 4.2 타입 좁히기 - 타입 가드

- 타입 좁히기: 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀 나가는 과정
- 타입 좁히기를 통해 더 정확하고 명시적인 타입 추론 가능
- 복잡한 타입을 작은 범위로 축소하여 타입 안정성 향상

### 타입 가드에 따라 분기 처리하기

- 타입스크립트에서의 분기 처리: 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따른 다른 동작을 수행하는 것
- 타입 가드: 런타임에 조건문을 사용하여 타입을 검사하고 타입을 좁혀주는 기능
  - 자바스크립트 연산자를 사용한 타입 가드: `typeof`, `instanceof`, `in`
  - 사용자 정의 타입 가드

### 원시 타입을 추론할 때: typeof 연산자 활용하기

```tsx
typeof A === B
```

- typeof는 자바스크립트 타입 시스템에만 대응 가능 → 복잡한 타입을 검증하는 데 한계가 있음
- typeof 연산자는 원시 타입을 좁히는 용도로 사용하는 것을 권장

#### typeof 연산자를 사용해서 검사할 수 있는 타입 목록

- string
- number
- boolean
- undefined
- object
- function
- bigint
- symbol

```tsx
const replaceHyphen: (date: string | Date) => string | Date = date => {
  if (typeof date === 'string') {
    return date.replace(/-/g, '/')
  }

  return date
}
```

### 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

```tsx
A instanceof B
```

- instanceof는 A의 프로토타입 체인에 생성자 B가 존재하는지 검사해서 존재 여부를 불리언 값으로 반환
  - A: 타입을 검사할 대상 변수 / B: 특정 객체의 생성자
- A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있음

```tsx
interface Range {
  start: Date
  end: Date
}

interface DatePickerProps {
  selectedDates?: Date | Range
}

const DatePicker = ({ selectedDates }: DatePickerProps) => {
  const [selected, setSelected] = useState(convertoToRange(selectedDates))
  // ...
}

export function convertToRange(selected?: Date | Range): Range | undefined {
  return selected instanceof Date // 인스턴스화된 객체 타입을 판별
    ? { start: selected, end: selected }
    : selected
}
```

### 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

```tsx
A in B
```

- A라는 속성이 B 객체에 존재하는지를 검사한 후 불리언 값 반환 → 속성 여부에 따라 객체 타입 구분 가능
- 프로토타입 체인으로 접근할 수 있는 속성이면 전부 true를 반환
- B 객체에 존재하는 A 속성에 undefined를 할당한다고 해서 false를 반환하지 않음 → delete로 속성 자체를 제거해야 false 반환
- 자바스크립트의 in 연산자는 런타임의 값만 검사하지만, 타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사함

> [!NOTE]
>
> 여러 객체 타입을 유니온 타입으로 가지고 있을 때 in 연산자를 사용해서 속성의 유무에 따라 조건 분기를 할 수 있다.

```tsx
interface BaseNoticeDialogProps {
  notiecTitle: string
  noiticeBody: string
}

interface NoticeDialogWithCookieProps extends BaseNoticeDialogProps {
  cookieKey: string
  noForADay?: boolean
  neverAgain?: boolean
}

export type NoticeProps = BaseNoticeDialogProps | NoticeDialogWithCookieProps

const NoticeDialog: React.FC<NoticeDialogProps> = props => {
  if ('cookieKey' in props) return <NoticeDialogWithCookie {...props} />
  return <NoticeDialogBase {...props} />
}
```

### is 연산자로 사용자 정의 타입 가드 만들어 활용하기

```tsx
A is B
```

- A : 매개변수 이름 / B : 타입
- 반환 타입이 타입 명제인 함수를 정의하여 사용하며 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급
- 타입 명제: 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수
- [문제로 풀어보기](https://typescript-exercises.github.io/#exercise=4)

```tsx
type User = {
  type: 'user'
  name: string
  age: number
  occupation: string
}

type Admin = {
  type: 'admin'
  name: string
  age: number
  role: string
}

export type Person = User | Admin

export function isAdmin(person: Person): person is Admin {
  return person.type === 'admin'
}

export function isUser(person: Person): person is User {
  return person.type === 'user'
}

export function logPerson(person: Person) {
  let additionalInformation: string = ''
  if (isAdmin(person)) {
    additionalInformation = person.role
  }
  if (isUser(person)) {
    additionalInformation = person.occupation
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`)
}
```

## 4.3 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

### 에러 정의하기

#### 에러 타입(텍스트 에러, 토스트 에러, 얼럿 에러) 정의

```tsx
type TextError = {
  errorCode: string
  errorMessage: string
}

type ToastError = {
  errorCode: string
  errorMessage: string
  toastShowDuration: number
}

type AlertError = {
  errorCode: string
  errorMessage: string
  onConfirm: () => void
}

type ErrorFeedBackType = TextError | ToastError | AlertError
```

#### 식별할 수 있는 유니온이 필요한 이유

```tsx
{
	errorCode: '999'.
	errorMessage: '잘못된 에러',
	toastShowDuration: 3000,
	onConfirm: () => {}
}
```

> 자바스크립트는 덕 타이핑 언어이기 때문에 ToastError의 toastShowDuration 필드와 AlertError의 onConfirm 필드를 모두 가지는 객체에 타입 에러를 뱉지 않는다.

### 식별할 수 있는 유니온

- 에러 타입을 구분할 방법이 필요
- 타입들이 서로 포함 관계를 가지지 않도록 정의하는 것이 필요 → 식별할 수 있는 유니온 활용

> [!NOTE]
>
> 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함 관계를 제거하는 것

#### 판별자 errorType 필드 정의

```tsx
type TextError = {
  type: 'TEXT'
  errorCode: string
  errorMessage: string
}

type ToastError = {
  type: 'TOAST'
  errorCode: string
  errorMessage: string
  toastShowDuration: number
}

type AlertError = {
  type: 'ALERT'
  errorCode: string
  errorMessage: string
  onConfirm: () => void
}
```

### 식별할 수 있는 유니온의 판별자 선정

#### 식별할 수 있는 유니온을 사용할 때 주의 사항

- 식별할 수 있는 유니온의 판별자는 유닛 타입으로 선언되어야 정상적으로 동작함
- 유닛 타입: 오직 하나의 정확한 값을 가지는 타입
  - null, undefined, 리터럴 타입, true, 1 등 정확한 값을 나타내는 타입이 유닛 타입에 해당
  - void, string, number와 같은 타입은 유닛 타입에 해당하지 않음

#### 식별할 수 있는 유니온 판별자로 사용할 수 있는 타입 (공식)

- 리터럴 타입
- 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화할 수 있는 타입은 포함되지 않음

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- 모든 케이스에 대해 철저하게 타입을 검사하는 것 → 모든 케이스에 대한 타입 검사 강제
- 모든 케이스에 대해 분기 처리를 해야만 유지보수 측면에서 안전하다고 생각되는 상황에 사용
- 예상치 못한 런타임 에러를 방지하거나 요구 사항이 변경 되었을 때 생길 수 있는 위험성을 줄일 수 있음

```tsx
type ProductPrice = '10000' | '20000' | '5000'

const getProductName = (productPrice: ProductPrice): string => {
  if (ProductPrice === '1000') return '배민상품권 1만원'
  if (ProductPrice === '2000') return '배민상품권 2만원'
  if (ProductPrice === '5000') return '배민상품권 5천원'
  else {
    exhaustiveCheck(productPrice)
    return '배민상품권'
  }
}

const exhaustiveCheck = (param: never) => {
  throw new Error('type error.')
}
```
