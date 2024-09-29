# 3장 고급 타입

## 3.1 타입스크립트만의 독자적 타입 시스템

![alt text](https://blog.kakaocdn.net/dn/Ay21n/btrBGNyrF3h/kef4nFIwdrXCG7gCKDsdm1/img.png)

### any 타입

- 자바스크립트에 존재하는 모든 값을 오류 없이 받을 수 있음
- 타입스크립트는 동적 타이핑 특징을 가진 자바스크립트에 정적 타이핑을 적용하는 것이 주된 목적
- any 타입은 이러한 목적을 무시하고 자바스크립트의 동적 타이핑으로 돌아가는 것과 비슷한 결과를 가져옴
- 따라서 any 타입을 변수에 할당하는 것은 지양해야 할 패턴

#### 타입스크립트에서 any 타입을 어쩔 수 없이 사용해야 하는 경우

- 개발 단계에서 임시로 값을 지정해야 할 때
- 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
  - API 요청 및 응답 처리
  - 콜백 함수 전달
  - 타입이 잘 정제되지 않아 파악이 힘든 외부 라이브러리
- 값을 예측할 수 없을 때 암묵적으로 사용

### unknown 타입

- any 타입과 유사하게 모든 타입의 값이 할당 가능
- any를 제외한 다른 타입으로 선언된 변수에 unknown 타입 값을 할당할 수 없음

| any                                                  | unknown                                                |
| ---------------------------------------------------- | ------------------------------------------------------ |
| 어떤 타입이든 any 타입에 할당 가능                   | 어떤 타입이든 unknown 타입에 할당 가능                 |
| any 타입은 어떤 타입으로도 할당 가능(단, never 제외) | unknown 타입은 any 타입 외에 다른 타입으로 할당 불가능 |

#### unknown 타입이 추가된 이유

- 할당 시점에서는 에러가 발생하지 않지만, 실행 시에는 에러 발생
- unknown 타입으로 할당된 변수는 어떤 값이든 올 수 있음을 의미하는 동시에 개발자에게 엄격한 타입 검사를 강제하는 의도

#### any 타입보다 unknown 타입 사용을 권장하는 이유

- any 타입과 유사하지만 타입 검사를 강제하고 타입이 식별된 후에 사용할 수 있기 때문에 any 타입보다 더 안전함
- 따라서 데이터 구조를 파악하기 힘들 때 any 타입 대신 unknown 타입으로 대체해서 사용하는 방법 권장

### void 타입

- 함수가 어떤 값을 반환하지 않는 경우에는 void를 지정하여 사용
- 함수 내부에 별도 반환문이 없다면 타입스크립트 컴파일러가 알아서 함수 타입을 void로 추론

### never 타입

- 값을 반환할 수 없는 타입
- 모든 타입의 하위 타입
- 즉, never 자신을 제외한 어떤 타입도 never 타입에 할당될 수 없음

#### 자바스크립트에서 값을 반환할 수 없는 경우

- 에러를 던지는 경우 → 에러를 발생시키는 경우 값을 반환하는 것으로 간주하지 않음

```tsx
function generateError(res: Response): never {
  throw new Error(res.getMessage())
}
```

- 무한히 함수가 실행되는 경우 → 무한 루프는 결국 함수가 종료되지 않음을 의미하기 때문에 값을 반환할 수 없음

```tsx
function checkStatus(): never {
  while (true) {
    console.log('check')
  }
}
```

### Array 타입

- `Object.prototype.toString.call()` 함수로 배열 타입을 확인할 수 있음

```tsx
const arr = []
console.log(Object.prototype.toString.call(arr)) // [object Array]
```

#### typeof를 사용하여 타입을 알 수도 있는데 `call` 함수를 사용하는 이유

- typeof의 경우 객체 타입을 단순히 object 타입으로 알려줌
- `Object.prototype.toString.call()` 함수는 객체의 인스턴스까지 알려줌

#### 자바스크립트에서도 확인할 수 있는 자료형인데도 타입스크립트에서 다시 배열을 다루는 이유

- 자바스크립트에서는 배열을 단독으로 배열이라는 자료형에 국한하지 않음
- 타입스크립트에서 Array라는 타입을 사용하기 위해서는 타입스크립트의 특수한 문법을 함께 다뤄야 함

#### 타입스크립트의 배열 타입

- 정적 타입의 특성을 살려 명시적인 타입을 선언하여 해당 타입의 원소 관리를 강제
- 선언 방식
  - 자료형 + 대괄호([]) 형식: `number[]`
  - 제네릭: `Array<number>`
- 다양한 자료형의 원소를 함께 다루는 배열 타입

```tsx
const array1: Array<number | string> = [1, 'string']
const array2: number[] | string[] = [1, 'string']
const array3: (number | string)[] = [1, 'string']
```

#### 튜플

- 기존 타입스크립트 배열 기능에 길이 제한까지 추가한 타입 시스템

```tsx
let tuple: [number, string, boolean] = [1, 'string', true]
```

- 배열 원소의 자리마다 명확한 의미를 부여해야 하는 경우 → `useState` 훅

```tsx
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>]
```

- 튜플과 배열의 성질을 혼합해서 사용 가능

```tsx
const httpStatusFromPaths: [number, string, ...string[]] = [
  400,
  'Bad Request',
  '/users/:id',
  '/users/:userId',
  '/users/:uuid',
]
```

- 옵셔널 프로퍼티(선택적 속성)를 명시하고 싶다면 물음표(?) 기호와 함께 선언

```tsx
const optionalTuple1: [number, number, number?] = [1, 2]
```

### enum 타입

- 타입스크립트에서 지원하는 특수한 타입
- 일종의 구조체를 만드는 타입 시스템
- enum을 사용해서 열거형을 정의할 수 있으며, 열거형은 각각의 멤버를 가지고 있음
- 각 멤버의 값을 스스로 추론하며, 기본적인 추론 방식은 숫자 0부터 1씩 늘려가며 값을 할당

```tsx
enum ProgrammingLanguage {
  Typescript, // 0
  JavaScript, // 1
  Java, // 2
  Python, // 3
  Kotlin, // 4
  Rust, // 5
  Go, // 6
}
```

- 값에 접근하는 방식은 자바스크립트에서 객체의 속성에 접근하는 방식과 동일하며, 역방향도 접근 가능

```tsx
ProgrammingLanguage.Typescript // 0
ProgrammingLanguage.Rust // 5
ProgrammingLanguage['Go'] // 6
ProgrammingLanguage[2] // 'Java'
```

- 각 멤버에 명시적으로 값을 할당 가능

```tsx
enum ProgrammingLanguage {
  Typescript = 'Typescript',
  JavaScript = 'Javascript',
  Java = 300,
  Python = 400,
  Kotlin, // 401
  Rust, // 402
  Go, // 403
}
```

- 주로 문자열 상수를 생성하는 데 사용되며, 이를 통해 응집력 있는 집합 구조체를 만들 수 있음

```tsx
enum ItemStatusType {
  DELIVERY_HOLD = 'DELIVERY_HOLD',
  DELIVERY_READY = 'DELIVERY_READY',
  DELIVERING = 'DELIVERING',
  DELIVERED = 'DELIVERED',
}

const checkItemAvailable = (itemStatus: ItemStatusType) => {
  switch (itemStatus) {
    case ItemStatusType.DELIVERY_HOLD:
    case ItemStatusType.DELIVERY_READY:
    case ItemStatusType.DELIVERING:
      return false
    case ItemStatusType.DELIVERED:
    default:
      return true
  }
}
```

#### `itemStatus` 타입이 문자열로 지정된 경우와 비교한 장점

- 타입 안정성: 타입에 명시되지 않은 다른 문자열은 인자로 받을 수 없어 타입 안정성이 우수함
- 명확한 의미 전달과 높은 응집력
- 가독성: 열거형 멤버를 통해 어떤 상태를 나타내는 지 쉽게 이해할 수 있음

#### 열거형 사용 시 주의 사항

- 숫자로만 이루어져 있거나 타입스크립트가 자동으로 추론한 열거형은 안전하지 않은 결과를 낳을 수 있음
- 역방향으로도 접근할 수 있기 때문에 할당된 값을 넘어서는 범위로 역방향을 접근하더라도 타입스크립트가 막지 않음
- 이러한 동작을 막기 위해 `const enum`으로 열거형을 선언하기도 함

```tsx
ProgrammingLanguage[200] // undefined를 출력하지만 별다른 에러를 발생시키지 않음

const enum ProgrammingLanguage {}
```

- const enum으로 열거형을 선언하더라도 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못함
- 반면 문자열 상수 방식으로 선언한 열거형은 미리 선언하지 않은 멤버로 접근을 방지

> 문자열 상수 방식으로 열거형을 사용하는 것이 숫자 방식보다 더 안전하며 의도하지 않은 값의 할당이나 접근을 방지하는데 도움이 된다.

- 열거형은 타입 공간과 값 공간에서 모두 사용되기 때문에 즉시 실행 함수(IIFE) 형식으로 변환됨
- 일부 번들러의 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있음
- 불필요한 코드의 크기가 증가하는 결과를 초래할 수 있음

> 이러한 문제를 해결하기 위해 `const enum` 또는 `as const` assertion을 사용해서 유니온 타입으로 열거형과 동일한 효과를 얻는 방법이 있다.

## 3.2 타입 조합

### 교차 타입(Intersection)

- 여러 가지 타입을 결합하여 하나의 단일 타입으로 만드는 방식 `A & B`

```tsx
type ProductItem = {
  id: number
  name: string
  type: string
  price: number
  imageUrl: string
  quantity: number
}

type ProductItemWithDiscount = ProductItem & { discountAmount: number }
```

### 유니온 타입(Union)

- 두 개 이상의 타입 중 하나가 될 수 있는 타입으로 만드는 방식 `A | B`

```tsx
type CardItem = {
  id: number
  name: string
  type: string
  imageUrl: string
}

type PromotionEventItem = ProductItem | CardItem

const printPromotionItem = (item: PromotionEventItem) => {
  console.log(item.name)

  console.log(item.quantity) // CardItem에 quantity 프로퍼티가 없으므로 컴파일 에러 발생
}
```

### 인덱스 시그니처(Index Signatures)

- 특정 타입의 속성 이름은 알 수 없지만 속성값의 타입을 알고 있을 때 사용

```tsx
interface IndexSignatureEx {
  [key: string]: number
}
```

### 인덱스드 엑세스 타입(Indexed Access Types)

- 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용

```tsx
const PromotionList = [
  { type: 'product', name: 'chicken' },
  { type: 'product', name: 'pizza' },
  { type: 'card', name: 'cheer-up' },
]

type ElementOf<T> = (typeof T)[number]
type PromotionItemType = ElementOf<PromotionList> // { type: string; name: string }
```

### 맵드 타입(Mapped Types)

- 다른 타입을 기반으로 한 타입을 선언할 때 사용
- 인덱스 시그니처 문법을 사용해서 반복적인 타입 선언을 효과적으로 줄일 수 있음

```tsx
type Example = {
  a: number
  b: string
  c: boolean
}

type Subset<T> = {
  [K in keyof T]?: T[K]
}

const aExample: Subset<Example> = { a: 3 }
const bExample: Subset<Example> = { b: 'hello' }
const cExample: Subset<Example> = { a: 4, c: true }
```

- 맵드 타입에서 매핑할 때는 `readonly`와 `?`를 수식어로 적용할 수 있음
  - `readonly`: 읽기 전용 수식어
  - `?`: 선택적 매개변수(옵셔널 파라미터) 전용 수식어
  - `-`: 기존 타입에 존재하던 `readonly`,`?`를 제거하는 수식어

```tsx
const BottomSheetMap = {
  RECENT_CONTACTS: RecentContactsBottomSheet,
  CARD_SELECT: CardSelectBottomSheet,
  SORT_FILTER: SortFilterBottomSheet,
  PRODUCT_SELECT: ProductSelectBottomSheet,
  REPLY_CARD_SELECT: ReplayCardSelectBottomSheet,
  RESEND: ResendBottomSheet,
  STICKER: StickerBottomSheet,
  BASE: null,
}

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap

type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID]: {
    resolver?: (payload: any) => void
    args?: any
    isOpened: boolean
  }
}
```

- 맵드 타입에서는 as 키워드를 사용하여 키를 재지정할 수 있음

```tsx
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
    resolver?: (payload: any) => void
    args?: any
    isOpened: boolean
  }
}
```

### 템플릿 리터럴 타입(Template Literal Types)

- 템플릿 리터럴 문자열을 사용하여 문자열 리터럴 타입을 선언할 때 사용

```tsx
type Stage = 'init' | 'select-image' | 'edit-image' | 'decorate-card' | 'capture-image'

type StageName = `${Stage}-stage`
// 'init-stage' | 'select-image-stage' | 'edit-image-stage' | 'decorate-card-stage' | 'capture-image-stage
```

### 제네릭(Generic)

- 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워 둔 다음에, 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정하여 사용하는 방식
- 함수, 타입, 클래스 등 여러 타입에 대해 하나하나 따로 정의하지 않아도 되기 때문에 재사용성이 크게 향상
- 타입 변수는 일반적으로 `<T>`와 같이 꺾쇠괄호 내부에 정의하며, 사용할 때는 함수에 매개변수를 넣는 것과 유사하게 원하는 타입을 넣어주면 됨
- 보통 타입 변수명으로 한 글자로 된 이름을 많이 사용

```tsx
type ExampleArrayType = T[]

const array1: ExampleArrayType<string> = ['치킨', '피자', '우동']
```

> [!NOTE]
>
> #### 제네릭이 any 타입과 다른 점
>
> - any를 사용하면 타입 검사를 하지 않고 모든 타입이 허용되는 타입으로 취급됨
> - 반면에 제네릭은 any처럼 아무 타입이나 무분별하게 받는 게 아니라, 배열 생성 시점에 원하는 타입으로 특정 가능
> - 즉, 제네릭을 사용하면 배열 요소가 전부 동일한 타입이라고 보장할 수 있음

- 타입 추론이 가능한 경우에는 타입 명시 생략 가능

```tsx
function exampleFunc<T>(arg: T): T[] {
  return new Array(3).fill(arg)
}

exampleFunc('hello') // string[]
```

- 제네릭은 일반화된 데이터 타입을 의미하므로 특정한 타입에서만 존재하는 멤버를 참조하려고 하면 안됨

```tsx
function exmapleFunc2<T>(arg: T): number {
  return arg.length // Error: Property 'length' does not exist on type 'T'
}
```

- 제약 조건을 설정하여 해결 가능

```tsx
interface TypeWithLength {
  length: number
}

function exmapleFunc2<T extends TypeWithLength>(arg: T): number {
  return arg.length
}
```

#### 제네릭을 사용 시 주의 사항

- 파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러 발생
- tsx가 타입스크립트 + JSX이므로 제네릭의 꺾쇠괄호와 태그의 꺾쇠괄호를 혼동하여 생기는 문제
- 제네릭 부분에 extends 키워드를 사용하여 컴파일러에게 특정 타입의 하위 타입만 올 수 있음을 확실히 알려주는 것으로 해결 가능

```tsx
// Error: JSX element 'T' has no corresponding closing tag
const arrowExmaple = <T,>(arg: T): T[] => {
  return new Array(5).fill(arg)
}

const arrowExmaple = <T extends {}>(arg: T): T[] => {
  return new Array(5).fill(arg)
}
```

## 3.3 제네릭 사용법

### 함수의 제네릭

- 어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용할 수 있음

```tsx
function ReadOnlyRepository<T>(target: ObjectType<T> | EntitySchema<T> | string): Repository<T> {
  return getConnection('ro').getRepository(target)
}
```

### 호출 시그니처의 제네릭

- 호출 시그니처: 타입스크립트의 함수 문법으로 함수의 매개변수와 반환 타입을 미리 선언하는 것
- 호출 시그니처를 사용할 때 제네릭 타입을 어디에 위치시키는지에 따라 타입의 범위와 제네릭 타입을 언제 구체 타입으로 한정할지를 결정할 수 있음

```tsx
interface useSelectPaginationProps<T> {
  categoryAtom: RecoilState<number>
  filterAtom: RecoilState<string[]>
  sortAtom: RecoilState<SortType>
  fetcherFunc: (props: CommonListRequest) => Promise<DefaultResponse<ContentListResponse<T>>>
}

export type UseRequesterHookType = <RequestData = void, ResponseData = void>(
  baseURL?: string | Headers,
  defaultHeader?: Headers,
) => [RequestStatus, Requester<RequestData, ResponseData>]
```

### 제네릭 클래스

- 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스
- 제네릭 클래스를 사용하면 클래스 전체에 걸쳐 타입 매개변수가 적용됨
- 특정 메서드만을 대상으로 제네릭을 사용하려면 해당 메서드를 제네릭 메서드로 선언하면 됨

### 제한된 제네릭

- 타입 매개변수에 대한 제약 조건을 설정하는 기능

#### string 타입으로 제한된 제네릭

```tsx
type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never
  ? Partial<Record<Key, boolean>>
  : never
```

### 확장된 제네릭

- 제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수도 있음

```tsx
export class APIResponse<Ok, Err = string> {
  private readonly data: Ok | Err | null
  private readonly status: ResponseStatus
  private readonly statusCode: number | null

  constructor(data: Ok | Err | null, statusCode: number | null, status: ResponseStatus) {
    this.data = data
    this.status = status
    this.statusCode = statusCode
  }

  public static Success<T, E = string>(data: T): APIResponse<T, E> {
    return new this<T, E>(data, 200, ResponseStatus.SUCCESS)
  }

  public static Error<T, E = unknown>(init: AxiosError): APIResponse<T, E> {
    if (!init.response) {
      return new this<T, E>(null, null, ResponseStatus.CLIENT_ERROR)
    }

    if (!init.response.data?.result) {
      return new this<T, E>(null, init.response.status, ResponseStatus.SERVER_ERROR)
    }

    return new this<T, E>(init.response.data.result, init.response.status, ResponseStatus.FAILURE)
  }

  // ...
}

// 사용하는 쪽 코드
const fetchShopStatus = async (): Promise<APIResponse<IShopResponse | null>> => {
  // ...

  return (await API.get<IShopResponse | null>('/v1/main/shop', config)).map(it => it.result)
}
```

### 제네릭 예시

#### API 응답 값의 타입을 제네릭으로 지정

```tsx
export interface MobileApiResponse<Data> {
  data: Data
  statusCode: string
  statusMessage?: string
}

export const fetchPriceInfo = (): Promise<MobileApiResponse<PriceInfo>> => {
  const priceUrl = 'https: ~~~' // url 주소

  return request({
    method: 'GET',
    url: priceUrl,
  })
}

export const fetchOrderInfo = (): Promise<MobileApiResponse<Order>> => {
  const orderUrl = 'https: ~~~' // url 주소

  return request({
    method: 'GET',
    url: orderUrl,
  })
}
```

#### 제네릭을 굳이 사용하지 않아도 되는 타입

```tsx
type GType<T> = T
type RequirementType = 'USE' | 'UN_USE' | 'NON_SELECT'
interface Order {
  getRequirement(): GType<RequirementType>
}
```

- GType은 굳이 제네릭을 사용하지 않고 타입 매개변수를 그대로 선언하는 것과 같은 기능을 하고 있음

```tsx
type RequirementType = 'USE' | 'UN_USE' | 'NON_SELECT'
interface Order {
  getRequirement(): RequirementType
}
```

#### any 사용하기

- any를 사용하면 제네릭의 장점과 타입 추론 및 타입 검사를 할 수 있는 이점을 누릴 수 없게 됨

#### 가독성을 고려하지 않은 사용

- 복잡한 제네릭은 의미 단위로 분할해서 사용하는 것을 권장

#### 복잡한 제네릭 개선 전

```ts
ReturnType<
  Record<
    OrderType,
    Partial<
      Record<CommonOrderStatus | CommonReturnStatus, Partial<Record<OrderRoleType, string[]>>>
    >
  >
>
```

#### 복잡한 제네릭 개선 후

```tsx
type CommonStatus = CommonOrderStatus | CommonReturnStatus
type PartialOrderRole = Partial<Record<OrderRoleType, string[]>>
type RecordCommonOrder = Record<CommonStatus, PartialOrderRole>
type RecordOrder = Record<OrderType, Partial<RecordCommonOrder>>

ReturnType<RecordOrder>
```
