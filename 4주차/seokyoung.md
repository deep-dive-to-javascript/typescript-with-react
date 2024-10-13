## 1. 조건부 타입

#### 조건에 따라 출력 타입을 다르게 도출하기

```tsx
Condition ? A : B
```

#### 조건부 타입의 장점

- 중복되는 타입 코드를 제거
- 상황에 따라 적절한 타입을 얻을 수 있음

> 더욱 정확한 타입 추론이 가능하다.

### `extends`와 제네릭을 활용한 조건부 타입

```tsx
T extends U ? X : Y
```

#### `extends` 키워드가 활용되는 상황

- 타입을 확장할 때
- 타입을 조건부로 설정할 때
- 제네릭 타입에서 한정자 역할로 사용할 때

```tsx
interface Bank {
  financialCode: string
  companyName: string
  name: string
  fullName: string
}

interface Card {
  financialCode: string
  companyName: string
  name: string
  appCardType?: string
}

type payMethod<T> = T extends 'card' ? Card : Bank
type CardPayMethodType = PayMethod<'card'>
type BankPayMethodType = PayMethod<'bank'>
```

### 조건부 타입을 사용하지 않았을 때의 문제점

#### 계좌, 카드, 앱 카드 등 3가지 결제 수단 정보를 가져오는 API 엔드포인트

- 계좌, 카드, 앱카드에 대한 정보 처리

```text
계좌 정보 엔드포인트: www.baemin.com/baeminpay/.../bank
카드 정보 엔드포인트: www.baemin.com/baeminpay/.../card
앱 카드 정보 엔드포인트: www.baemin.com/baeminpay/.../appcard
```

#### 타입 정의

```tsx
/**
 * - 서버에서 받아오는 결제 수단 기본 타입
 * - 은행과 카드에 모두 들어가 있음
 */
interface PayMethodBaseFromRes {
  financialCode: string
  name: string
}

/**
 * - Bank: 결제 기본 수단 타입인 PayMethodBaseFromRes를 상속받아 구현된 은행 결제 수단 타입
 * - Card: 결제 기본 수단 타입인 PayMethodBaseFromRes를 상속받아 구현된 카드 결제 수단 타입
 */
interface Bank extends PayMethodBaseFromRes {
  fullName: string
}
interface Card extends PayMethodBaseFromRes {
  appCardType?: string
}

/**
 * - 최종적인 은행, 카드 결제 수단 타입
 * - 프론트에서 추가되는 UI 데이터 타입과 제네릭으로 받아오는 Bank 또는 Card 합성
 * - extends로 Bank 또는 Card를 포함하지 않는 타입이 제네릭으로 넘어오지 못하도록 방어
 * - 제네릭을 활용해서 중복된 코드 제거하고 하나의 타입으로 정의
 */
type PayMethodInfo<T extends BanK | Card> = T & PayMethodInterface

/**
 * - 프론트에서 관리하는 결제 수단 관련 데이터
 * - UI를 구현하는데 사용되는 타입
 */
type PayMethodInterface = {
  companyName: string
}
```

#### `useGetRegisteredList` 함수

> react-query의 `useQuery`를 사용하여 구현한 커스텀 훅

```tsx
// 최종적인 결제 수단 정보를 담은 타입
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>
// 결제 수단 타입(card, appcard, bank)을 인자로 받아, 해당 결제 수단의 정보를 반환하는 함수
export const useGetRegisterdList = (
  type: 'card' | 'appcard' | 'bank',
): UseQueryResult<PayMethodType[]> => {
  // API 호출 시 사용할 URL 설정 시 appcard일 경우 card로 변환
  const url = `baeminpay/codes/${type === 'appcard' ? 'card' : type}`

  // axios 래핑해서 사용하며, 서버에서 useType이 'USE'인 항목만 필터링하여 데이터를 가져오는 함수
  const fetcher = fetcherFactory<PayMethodType[]>({
    onSuccess: res => {
      const usablePocketList = res?.filter(
        (pocket: PocketInfo<Card> | PocketInfo<Bank>) => pocket?.userType === 'USE' ?? [],
      )
      return usablePocketList
    },
  })

  // useQuery를 래핑해서 사용하며, URL로부터 데이터 가져오는 함수
  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher)

  return result
}
```

> `useGetRegisteredList` 함수는 타입을 구분해서 넣는 사용자의 의도와는 다르게 정확한 타입을 반환하지 못하는 함수가 됐다.
>
> 인자로 넣는 타입에 알맞은 타입을 반환하고 싶지만, 타입 설정이 유니온으로만 되어있기 때문에 타입스크립트는 해당 타입에 맞는 Data 타입을 추론할 수 없다.

### extends 조건부 타입을 활용하여 개선하기

> 인자에 따라 반환되는 타입을 다르게 설정하고 싶다면 `extends`를 사용한 조건부 타입을 활용하면 된다.

#### 조건부 타입으로 `PayMethodType` 개선하기

```tsx
type PayMethodType<T extends 'card' | 'appcard' | 'bank'> = T extends 'card' | 'appcard'
  ? Card
  : Bank
```

#### 개선한 `PayMethodType`으로 `useGetRegisteredList` 리팩토링

```tsx
export const useGetRegisterdList = <T extends 'card' | 'appcard' | 'banck'>(
  type: T,
): UseQueryResult<PayMethodType<T>[]> => {
  const url = `baeminpay/codes/${type === 'appcard' ? 'card' : type}`

  const fetcher = fetcherFactory<PayMethodType<T>[]>({
    onSuccess: res => {
      const usablePocketList = res?.filter(
        (pocket: PocketInfo<Card> | PocketInfo<Bank>) => pocket?.userType === 'USE' ?? [],
      )
      return usablePocketList
    },
  })

  const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher)

  return result
}
```

> 이제 인자로 card 또는 appcard를 넣는다면 `PocketInfo<Card>`를 반환하고, bank를 넣는다면 `PocketInfo<Bank>`를 반환한다.
>
> 이로써 사용자는 `useGetRegisteredList` 함수를 사용할 때 불필요한 타입 가드를 하지 않아도 된다. 또한 데이터를 `PocketInfo<Card>`받는 컴포넌트의 props로 넘겨줄 때 불필요한 타입 단언을 하지 않아도 된다.

#### extends 활용 예시

- 제네릭과 `extends`를 함께 사용해 제네릭으로 받는 타입을 제한 → 휴먼 에러 방지
- `extends`를 활용해 조건부 타입 설정 → 불필요한 타입 가드, 타입 단언 등을 방지

### `infer`을 활용해서 타입 추론하기

- 타입을 추론하는 역할
- 삼항 연산자를 사용한 조건문 형태
  - `extends`로 조건 서술
  - `infer`로 타입 추론

#### `UnpackPromise` 타입

> 제네릭으로 T를 받아 T가 Promise로 래핑된 경우라면 K를 반환하고, 그렇지 않은 경우에는 any를 반환한다.
>
> → `Prosmise<infer K>는 Promise의 반환 값을 추론해 해당 값의 타입을 K로 한다는 의미

```tsx
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any

const promises = [Promise.resolve('Mark'), Promise.resolve(38)]
type Expected = UnpackPromise<typeof promises> // string | number
```

## 2. 템플릿 리터럴 타입 활용하기

### 템플릿 리터럴 타입

- 자바스크립트의 템플릿 리터럴 문법을 사용해 특정 문자열에 대한 타입을 선언할 수 있는 기능

```tsx
type HeadingNumber = 1 | 2 | 3 | 4 | 5
type HeaderTag = `h${HeadingNumber}`
```

#### 템플릿 리터럴 타입 사용 시 주의 사항

- 유니온을 추론하는 데 시간이 오래 걸리면 비효율적이기 때문에 타입스크립트가 타입을 추론하지 않고 에러를 내뱉는 경우가 있음
- 템플릿 리터럴 타입에 삽입된 유니온 조합의 경우의 수가 너무 많지 않게 적절하게 나누어 타입을 정의하는 것이 좋음

```tsx
type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
type Chunk = `${Digit}${Digit}${Digit}${Chunk}`
type PhoneNumberType = `010-${Chunk}-${Chunk}
```

## 3. 커스텀 유틸리티 타입 활용하기

### 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

#### 유틸리티 타입 `Pick`으로 중복 없이 `StyledProps` 정의

```tsx
// HrComponent.tsx
export type Props = {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
};

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  );
};

// style.ts
import { Props } from '../HrComponent.tsx';
type StyledProps = Pick<Props, "height" | "color" | "isFull">;
const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) = > height || "10px"};
  margin: 0;
  background-color: ${({ color }) = > colors[color || "gray7"]};
  border: none;
  ${({ isFull }) => isFull && css`
    margin: 0 -15px;
  `}
`;
```

### `PickOne` 유틸리티 함수

> 타입스크립트에는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 있다.

```tsx
type Card = {
  card: string
}
type Account = {
  account: string
}

function withdraw(type: Card | Account) {}
withdraw({ card: 'hyundai', account: 'hana' })
```

> [!NOTE]
>
> #### 왜 타입 에러가 발생하지 않을까?
>
> - 집합 관점으로 볼 때 유니온은 합집합이 되기 때문
> - card, account 속성이 모두 포함되어도 합집합의 범주에 들어가기 때문에 타입 에러가 발생하지 않는 것

#### 식별할 수 있는 유니온으로 객체 타입을 유니온으로 받기

- 각 타입에 type이라는 공통된 속성을 추가하여 구분짓는 방법

```tsx
type Card = {
  type: 'card'
  card: string
}
type Account = {
  type: 'account'
  account: string
}

function withdraw(type: Card | Account) {}
withdraw({ type: 'card', card: 'hyundai' })
withdraw({ type: 'account', account: 'hana' })
```

> 식별할 수 있는 유니온으로 문제를 해결할 수 있지만 일일이 type을 다 넣어줘야 하는 불편함이 생긴다.

#### `PickOne` 커스텀 유틸리티 타입 구현하기

- 구현하고자 하는 타입: account 또는 card 속성 하나만 존재하는 객체를 받는 타입
- account일 때는 card를 받지 못하고, card일 때는 account를 받지 못하게 하려면 하나의 속성이 들어왔을 때 다른 타입을 옵셔널한 undefined 값으로 지정하는 방법

```tsx
type PayMethod =
  | { account: string; card?: undefined; payMoney?: undefined }
  | { account?: undefined; card: string; payMoney?: undefined }
  | { account?: undefined; card?: undefined; payMoney: string }
```

> 결국 선택하고자 하는 하나의 속성을 제외한 나머지 값을 `옵셔널 + undefined`로 설정하면 원하고자 하는 속성만 받도록 구현할 수 있다.

```tsx
type PickOne<T> = {
  [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>
}[keyof T]
```

#### `PickOne` 살펴보기

- `One<T>`: 전달받은 객체를 그대로 반환하는 타입
- `ExcludeOne<T>`: 전달받은 객체에 모든 속성을 옵셔널로 반환하는 타입
- `PickOne<T`>: 전달받은 T 타입의 1개의 키만 값을 가지고 나머지 키는 옵셔널한 undefined 값을 가진 객제로 반환하는 타입

```tsx
type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];

type ExcludeOne<T> = {
  [P in keyof T]: Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];
const excludeone: ExcludeOne<Card> =

type PickOne<T> = One<T> & ExcludeOne<T>;
```

#### `PickOne` 적용하기

```tsx
type Card = { card: string }
type Account = { account: string }
type CardOrAccount = PickOne<Card & Account>

function withdraw(type: CardOrAccount) {}
withdraw({ card: 'hyndai' }) // (O)
withdraw({ account: 'hana' }) // (O)
withdraw({ card: 'hyundai', account: undefined }) // (O)
withdraw({ card: undefined, account: 'hana' }) // (O)
withdraw({ card: 'hyundai', account: 'hana' }) // 에러 발생
```

### `NonNullable` 타입 검사 함수를 사용하여 간편하게 타입 가드하기

#### `NonNullable` 타입

- 타입스크립트에서 제공하는 유틸리티 타입
- 제네릭으로 받는 T가 null 또는 undefined일 때ㅑ, never 또는 T를 반환하는 타입
- `NonNullable`을 사용하면 null이나 undefined가 아닌 경우를 제외할 수 있음

```tsx
type NonNullable<T> = T extends null | undefined ? never : T
```

#### `NonNullable` 함수

- `NonNullable` 유틸리티 타입을 사용하여 null 또는 undefined를 검사하는 타입 가드 함수
- 매개변수가 null 또는 undefined일 때, false를 반환
- `is` 키워드가 사용되어 `NonNullable` 함수를 사용하는 쪽에서 true가 반환된다면 넘겨준 인자는 null이나 undefined가 아닌 타입으로 타입 가드 됨

```tsx
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined
}
```

#### `Promise.all`을 사용할 때 `NonNullable` 적용하기

```tsx
class AdCampaignAPI {
  static async operating(shopNo: number): Promise<AdCampaign[]> {
    try {
      return await fetch(`/ad/shopNumber=${shopNo}`)
    } catch (error) {
      return null
    }
  }
}
```

```tsx
const shopList = [
	{shopNo: 100, category: 'chicken'},
	{shopNo: 101, category: 'pizza'},
	{shopNo: 102, category 'noodle'}
];

const shopAdCampaignList = await Pormise.all(
	shopList.map((shop)=> AdCampaignAPI.operating(shop.shopNo))
);

const shopAds = shopAdCampaignList.filter(NonNullable);
```

## 4. 불변 객체 타입으로 활용하기

```tsx
const colors = {
  red: '#F45452',
  green: '#0c9252a',
  blue: '#1A7cff',
}

const getColorHex = (key: string) => colors[key] // any
```

#### `keyof` 연산자로 객체의 키값을 타입으로 추출하기

- 객체 타입을 받아 해당 객체의 키값을 string 또는 number의 리터럴 유니온 타입 반환

```tsx
interface ColorType {
  red: string
  green: string
  blue: string
}

type ColorKeyType = keyof ColorType // 'red' | 'green' | 'blue'
```

#### `typeof` 연산자로 값을 타입으로 다루기

```tsx
const colors = {
	red: '#F45452',
	green: '#0c9252a'
	blue: '#1A7cff'
}

type ColorsType = typeof colors
/*
{
	red: string
	green: string
	blue: string
}
*/
```

#### `keyof typeof`

```tsx
type Colors = keyof typeof colors // 'red' | 'green' | 'blue'
```

## 5. `Record` 원시 타입 키 개선하기

### 무한한 키를 집합으로 가지는 `Record`

```tsx
type Category = string
interface Food {
  name: string
  // ...
}

const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: '제육덮밥' }, { name: '뚝배기 불고기' }],
  일식: [{ name: '초밥' }, { name: '텐동' }],
}

foodByCategory['양식'] // Food[]로 추론
foodByCategory['양식'].map(food => console.log(food.name)) // 컴파일 단계에서 오류가 발생하지 않고 런타임에서 undefined로 오류 반환
```

> 자바스크립트의 옵셔널 체이닝으로 런타임 에러 방지할 수 있지만 어떤 값이 undefined인지 매번 판단해야 하는 번거로움이 생긴다.

```tsx
foodByCategory['양식']?.map(food => console.log(food.name))
```

### 유닛 타입으로 변경하기

- 키가 유한한 집합이라면 유닛 타입을 사용할 수 있음
- 키가 무한해야 하는 상황에는 적합하지 않음

```tsx
type Category = '한식' | '일식'
```

### `Partial`을 활용하여 정확한 타입 표현하기

- 키가 무한한 상황에서 `Partial`을 이용하여 해당 값이 undefined일 수 있는 상태임을 표현할 수 있음
- 개발자는 타입스크립트의 안내를 보고 옵셔널 체이닝을 사용하거나 조건문을 사용하는 등 사전 조치 가능 → 예상치 못한 런타임 오류 줄일 수 있음

```tsx
type PartialRecord<K extends string, T> = Partial<Record<K, T>>
type Category = string
```
