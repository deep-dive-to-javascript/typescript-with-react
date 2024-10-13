## 5장 타입 활용하기
### 5.1 조건부 타입
- 타입스크립트에서는 조건부 타입을 사용해 조건에 따라 출력 타입을 다르게 도출할 수 있음

1. extends와 제네릭을 활용한 조건부 타입
```ts
T extends U ? X : Y
```
- 조건부 타입에서 extends를 사용할 때는 자바스크립트 삼항 연산자와 함께 씀

2. 조건부 타입을 사용하지 않았을 떄의 문제점 

조건부 타입을 사용하지 않으면 다음과 같은 문제점이 발생할 수 있다
```text
계좌 정보 엔드포인트: www.baemin.com/baeminpay/.../bank
카드 정보 엔드포인트: www.baemin.com/baeminpay/.../card
앱 카드 정보 엔드포인트: www.baemin.com/baeminpay/.../appcard
```

함수를 구현하기 앞서 사용되는 타입들을 서버에서 받아오는 타입, UI 관련 타입 그리고 결제 수단별 특성에 따라 구분하였다.
```ts
// 서버에서 받아오는 결제 수단 기본 타입으로 은행과 카드에 모두 들어가 있다.
interface PayMethodBaseFromRes {
	financialCode: string;
	name: string;
}

// 은행과 카드 각각에 맞는 결제 수단 타입이다. 결제 수단 기본 타입인 PayMethodBaseFromRes를 상속받아 구현한다.
interface Bank extends PayMethodBaseFromRes {
	fullName: string;
}
interface Card extends PaymethodBaseFromRes {
	appCardType?: string;
}

/*
	최종적인 은행, 카드 결제 수단 타입이다. 프론트에서 추가되는 UI 데이터 타입과 제네릭으로 받아오는 Bank 또는 Card를 합성한다.
	extends를 제네릭에서 한정자로 사용하여 Bank또는 Card를 포함하지 않는 타입은 제네릭으로 넘겨주지 못하게 방어한다.
	BankPayMethodInfo = PayMethodInterface & Bank처럼 카드와 은행의 타입을 만들어줄 수 있지만 제네릭을 활용해서 중복된 코드를 제거한다.
*/
type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;

//프론트에서 관리하는 결제 수단 관련 데이터로 UI를 구현하는 데 사용되는 타입이다.
type PayMethodInterface = {
	companyName: string;
	// ...
}
```
이제 react-query의 useQuery를 사용하여 구현한 커스텀 훅인 useGetRegisteredList 함수를 살펴보자

```ts
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;
// useGetRegisteredList 함수는 useQuery의 반환값을 돌려준다
export const useGetRegisteredList = (
type: 'card' | 'appcard' | 'bank'
): useQueryResult<PayMethodType[]> => {
const url = `baeminpay/code/${type === 'appcard' ? 'card' : type}`;

    // fetcherFactory는 axios를 래핑해주는 함수이며, 서버에서 데이터를 받아온 후 onSuccess 콜백 함수를 거친 결과값을 반환한다
	const fetcher = fetcherFactory<PayMethodType[]>({
		onSuccess: (res) => {
			const usablePockerList = res?.filter(
				(pocket: PocketInfo<Card> | PocketInfo<Bank>) => pocket?.useType === 'USE'
			) ?? [];
			
			return usablePockerList;
		}
	});
    
	const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
	return result;
}
```

반환 타입 설정이 유니온으로만 되어있기 때문에 타입스크립트는 해당 타입에 맞는 Data 타입을 추론할 수 없다. 

반환되는 타입을 다르게 설정하고 싶다면 extends를 사용한 조건부 타입을 활용하면 된다.

3. extends 조건부 타입을 활용하여 개선하기 

useGetRegisteredList 함수의 반환 Data는 인자 타입에 따라 정해져 있다. <br/>
타입으로 ‘card’ 또는 ‘appcard’를 받으면 카드 결제 수단 정보 타입인 PocketInfo를 반환하고, ‘bank’를 받는다면 PocketInfo를 반환한다.

```text
type: 'card' | 'appcard' -> PocketInfo<Card>
type: 'bank' -> PocketInfo<Bank>
```

조건부 타입을 사용해서 PayMethodInfo | PayMethodInfo 타입이었던 PayMethodType 타입을 개선해보자

```ts
type PayMethodType<T extends 'card' | 'appcard' | 'bank'> = T extends 'card' | 'appcard' ? Card : Bank;
```

이제 useGetRegisteredList 함수를 개선해보자

```ts
export const useGetRegisteredList = <T extends 'card' | 'appcard' | 'bank'>(
	type: T
): useQueryResult<PayMethodType<T>[]> => {
	const url = `baeminpay/code/${type === 'appcard' ? 'card' : type}`;
	
	const fetcher = fetcherFactory<PayMethodType<T>[]>({
		onSuccess: (res) => {
			const usablePockerList = res?.filter(
				(pocker: PocketInfo<Card> | PocketInfo<Bank>) => pocket?.useType === 'USE'
			) ?? [];
			
			return usablePockerList;
		}
	});
	
	const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher);
	return result;
}
```

조건부 타입을 활용하여 PayMethodType이 사용자가 인자에 넣는 타입 값에 맞는 타입만을 반환하도록 구현했다. <br/>
이로써 사용자는 useGetRegisteredList 함수를 사용할 때 불필요한 타입 가드를 하지 않아도 된다.

4. infer를 활용해서 타입 추론하기
- infer는 조건부 타입에서 사용되는 키워드로, 타입을 추론할 때 사용된다.

infer를 활용한 예시를 살펴보자
```ts
type UnpackPromise<T> = T extends Promise<infer K> ? K : any;
```

UnpackPromise는 T가 Promise로 래핑된 경우라면 K를 반환하고 아니라면 any를 반환한다. Promise의 반환 값을 추론해 해당 값의 타입을 K로 한다는 의미이다.

```ts
const promises = [Promise.resolve('Hello'), Promise.resolve(123)];
type expected = UnpackPromise<typeof promises>; // string | number
```

아래는 배민 라이더를 관리하는 라이더 어드민 서비스에서 사용하는 타입이다.

```ts
interface RouteBase {
	name: string;
	path: string;
	component: ComponentType;
}

export interface RouteItem {
	name: string;
	path: string;
	component?: ComponentType;
	pages?: RouteBase[];
}

export const routes: RouteItem[] = [
	{
			name: "기기 내역 관리",
			path: "/device-history",
			component: DeviceHistoryPage
	},
	{
			name: "헬멧 인증 관리",
			path: "/helmet-certification",
			component: HelmetCertificationPage
	},
]
```
RouteBase와 RouteItem은 라이더 어드민에서 라우팅을 위해 사용하는 타입이다. <br/>
RouteItem의 name은 pages가 있을 때는 단순히 이름의 역할만 하며 그렇지 않을 때는 사용자 권한과 비교한다.

```ts
export interface SubMenu {
	name: string;
	path: string;
}

export interface MainMenu {
	name: string;
	path?: string;
	subMenus?: SubMenu[];
}

export type MenuItem = SubMenu | MainMenu;
export const menuList: MenuItem[] = [
	{
		name: "계정 관리",
		subMenus: [
			{
				name: "기기 내역 관리",
				path: "/device-history",
			},
			{
				name: "헬멧 인증 관리",
				path: "/helmet-certification",
			}
		]
	}
]
```

MainMenu와 SubMenu는 메뉴 리스트에서 사용하는 타입으로 권한 API를 통해 반환된 사용자 권한과 name을 비교하여 사용자가 접근할 수 있는 메뉴만 렌더링한다.<br/>
MainMenu의 name은 subMenus를 가지고 있을 때 단순히 이름의 역할만 하며, 그렇지 않을 때는 권한으로 간주된다.

여기서 하지만 name은 string 타입으로 정의되어 있기 때문에 routes와 menuList에서 subMenu의 기기 내역 관리처럼 서로 다른 값이 입력되어도 컴파일타임에서 에러가 발생하지 않는다.

```ts
type PermissionNames = '기기 정보 관리' | '안전모 인증 관리' | '운행 여부 조회'; //...
```

이를 개선하기 위해 PermissionNames 처럼 변도 타입을 선언하여 name을 관리하는 방법도 있지만, 권한 검사가 필요 없는 subMenu나 pages가 존재하는 name을 따로 처리해야한다.<br/>
이때 infer와 불변 객체(as const)를 활용해서 menuList 또는 routes의 값을 추출하여 타입으로 정의하는 식으로 개선할 수 있다.

```ts
export interface MainMenu {
	// ...
	subMenu?: ReadonlyArray<SubMenu>;
}

// 불변객체로 정의
export const menuList = [
	// ...
] as const;

interface RouteBase {
	name: PermissionNames;
	path: string;
	component: ComponentType;
}

export type RouteItem =
	| {
			name: string;
			path: string;
			component?: ComponentType;
			pages: RouteBase[];
		}
	| {
			name: PermissionNames;
			path: string;
			component: ComponentType;
		}

```

조건에 맞는 값을 추출한 UnpackMenuNames라는 타입 추가
```ts
type UnpackMenuNames<T extends ReadonlyArray<MenuItem>> = T extends ReadonlyArray<infer U>
	? U extedns MainMenu
		? U['subMenus'] extends infer V
			? V extends ReadonlyArray<SubMenu> ? UnpackMenuNames<V> : U['name']
			: never
		: U extends SubMenu ? U['name'] : never
	:never;

export type PermissionNames = UnpackMenuNames<typeof menuList>; // PermissionNames = '기기 내역 관리' | '헬멧 인증 관리'
```

- U가 MainMenu인 경우 subMenus를 infer V로 추출한다.
- subMenus는 옵셔널한 타입이기 때문에 추출한 V가 존재한다면(SubMenu 타입에 할당할 수 있다면) UnpackMenuNames를 재귀적으로 호출한다.
- V가 존재하지 않는다면 MainMenu의 name은 권한에 해당하므로 U['name']을 반환한다.
- U가 MainMenu가 아니라 SubMenu에 할당할 수 있다면(U는 SubMenu 타입이기 때문에) U['name']을 반환한다.

### 5.2 템플릿 리터럴 타입 활용하기

- 타입스크립트에서는 유니온 타입을 사용하여 변수 타입을 특정 문자열로 지정할 수 있음
```ts
type HeaderTag = "h1" | "h2" | "h3" | "h4" | "h5";

```

- 타입스크립트 4.1부터 이를 확장하는 방법인 템플릿 리터럴 타입을 지원하기 시작

```ts
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;
```

- 수평 또는 수직 방향을 표현하는 Direction타입을 다음과 같이 표현할 수 있음

```ts
type Direction = 
| "top"
| "topLeft"
| "topRight"
| "bottom"
| "bottomLeft"
| "bottomRight";
```

- 템플릿 리터럴 타입을 사용하면 다음과 같이 표현할 수 있음

```ts
type Vertical = 'top' | 'bottom';
type Horizon = 'left' | 'right';

type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}}`;
```

> 타입스크립트 컴파일러가 유니온을 추론하는 데 시간이 오래 걸리면 비효율적이기 때문에 경우의 수가 너무 많지 않게 조절하는 것이 중요

### 5.3 커스텀 유틸리티 타입 활용하기
1. 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

먼저 유틸리티 타입을 활용하지 않으면 어떤 불편함이 발생하는지 살펴보자

Hr 컴포넌트 Props의 속성은 styled-components인 HrComponent에 바로 연결되며 타입도 역시 같다.

styles.ts주석 아래 코드처럼 styledProps타입을 새로 정의하여도 Props와 똑같은 타입임에도 새로 작성해야하며 불가피하게 중복된 코드가 생긴다.

```ts
export type Props = {
	height?:string;
	color?: keyof typeof colors;
	isFull?:boolean;
	className?:string;
}

export const Hr: VFC<Props> = ({height, color, isFull, className}) => {
	...
	return <HrComponent height={height} color={color} isFull={isFull} className={className} />;
};

//styles.ts
import {Props} from '...';
type StyledProps = Pick<Props, "height" | "color" | "isFull">;
const HrComponent = styled.hr<StyledProps>`
	height: ${({height}) => height || '10px'};
	margin: 0;
	background-color: ${({color}) => colors[color || 'gray']};
	border: none;
	${({isFull}) => isFull && css`
        margin: 0 -15px;
    `}
`;
```

이런 문제를 Pick, Omit과 같은 유틸리티 타입으로 개선할 수 있다.
```ts
type StyledProps = Pick<Props, "height" | "color" | "isFull">;
```

2. PickOne 유틸리티 함수
- 타입스크립트에서는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 있음
```ts
type Card = {
	card: string
};

type Account = {
	account: string
}

function withdraw(type: Card | Account){
	...
}

withdraw({card: 'hyundai', account: 'hana'});
```

집합관점으로 볼 때 유니온은 합집합이 되기 때문에 card, account 속성이 하나씩만 할당된 상태도 허용하지만 모두 포함되어도 에러가 발생하지 않는다. <br/>
이런 문제를 해결하기 위해 타입스크립트에서는 식별할 수 있는 유니온 기법을 자주 활용한다.

식별할 수 있는 유니온은 각 타입에 type이라는 공통된 속성을 추가하여 구분짓는 방법이다.

```ts
type Card = {
	type: "card";
	card: string;
}

type Account = {
	type: "account";
	account: string;
}

function withfraw(type: Card | Account){
	...
}

withdraw({type: 'card', card: 'hyundai'});
withdraw({type: 'account', account: 'hana'});
```

식별할 수 있는 유니온으로 문제를 해결할 수 있지만 일일이 type을 다 넣어줘야하는 불편함이 생긴다. 또한 이미 구현된 상태에서 식별할 수 있는 유니온을 적용하려면 해당 함수를 사용하는 부분을 모두 수정해야한다.

이러한 상황을 방지하기 위해 PickOne이라는 유틸리티 타입을 구현하여 적용해보자.

account일때는 card를 받지 못하고, card일때는 account를 받지 못하게 제약을 걸어주려면 하나의 속성이 들어왔을 때 다른 속성은 undefined로 만들어주면 된다.

```ts
type PickOne<T> = {
	[P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];
```

**PickOne 살펴보기**

앞의 유틸리티 타입을 하나씩 자세하게 뜯어보자. PickOne 타입을 2가지 타입으로 분리해서 생각할 수 있다.

- One<T>

```ts
type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];
```

1. [P in keyof T]에서 T는 객체로 가정하기 때문에 P는 T 객체의 키 값을 말한다.
2. Record<P, T[P]>는 P타입을 키로 가지고, value는 P를 키로 둔 T 객체의 값의 레코드 타입을 말한다.
3. 따라서{ [P in keyof T]: Record<P, T[P]> }에서 키는 T 객체의 키 모음이고, value는 해당 키의 원본 객체 T를 말한다.
4. 3번의 타입에서 다시 [keyof T]의 키 값으로 접근하기 때문에 최종 결과는 전달받은 T와 같다.

- ExcludeOne

```ts
type ExcludeOne<T> = { [P in keyof T]: Partial<Record<Exclude<keyof T, P>, undefined>> }[keyof T];
```

1. [P in keyof T]에서 T는 객체로 가정하기 때문에 P는 T 객체의 키값을 말한다.
2. Exclude<keyof T, P>는 T 객체가 가진 키값에서 P 타입과 일치하는 키값을 제외한다.
3. Record<A, undefined>는 키로 A 타입을, 값으로 undefined 타입을 갖는 레코드 타입이다. 즉, 전달 받은 객체 타입을 모두 { [key]: undefined } 형태로 만든다. 이 타입을 B라고 하자.
4. Partial<\B>는 B 타입을 옵셔널로 만든다. 따라서 { [key]?: undefined }와 같다.
5. 최종적으로 [P in keyof T]로 매핑된 타입에서 동일한 객체의 키값인 [keyof T]로 접근하기 때문에 4번 타입이 반환된다.

- PickOne

```ts
type PickOne<T> = One<T> & ExcludeOne<T>;
```

1. One<T>와 ExcludeOne<T>를 합친다.
2. One<T>는 전달받은 객체 T와 같은 타입이다.
3. ExcludeOne<T>는 전달받은 객체 T의 키값을 모두 옵셔널로 만든 타입이다.
4. 따라서 최종적으로 One<T>와 ExcludeOne<T>의 교집합이 반환된다.
5. One<T>는 전달받은 객체 T와 같은 타입이므로 ExcludeOne<T>의 옵셔널 속성은 전달받은 객체 T의 키값 중 하나만 존재하게 된다.

이제 PickOne을 사용하여 PickOne<Card & Account>를 사용하면 card와 account 중 하나만 받을 수 있는 타입을 정의할 수 있다.

```ts
type Card = {
    type: "card";
    card: string;
}

type Account = {
    type: "account";
    account: string;
}

type PickOne<T> = {
    [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];

function withdraw(type: PickOne<Card & Account>){
    ...
}

withdraw({ card: 'hyundai', account: 'haha' }); // 에러 발생
```

3. NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

일반적으로 if문을 사용해서 null 처리 타입 가드를 적용하지만, is 키워드와 NonNullable 타입으로 타입 검사를 위한 유틸 함수를 만들어서 사용할 수도 있다.

NonNullable 타입이란 타입스크립트에서 제공하는 유틸리티 타입으로 제네릭으로 받는 T가 null 또는 undefined일 때 never 또는 T를 반환하는 타입이다. 

NonNullable을 사용하면 null이나 indefined가 아닌 경우를 제외할 수 있다.

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```

NonNullable 함수는 매개변수인 value가 null 또는 undefined라면 false를 반환한다. 

is 키워드가 쓰였기 때문에 NonNullable 함수를 사용하는 쪽에서 true가 반환된다면 넘겨준 인자는 null이나 undefined가 아닌 타입으로 타입 가드가 된다.

```ts
function NonNullable<T>(value: T) = value is NonNullable<T> {
	return value !== null && value !== undefined;
}
```

아래는 각 상품 광고를 노출하는 api함수 레이어다. 여러 상품을 조회할 때 하나의 API에서 에러가 발생한다고 해서 전체 광고가 보이지 않으면 안 된다. 

따라서 try-catch문을 사용하여 에러가 발생할 때는 null을 반환하고 있다.

```ts
class AdCampaignAPI {
	static async operating(shopNo: number): Promise<AdCampaign[]>{
		try{
			return await fetch(`/ad/shopNumber=${shopNo}`);
		}catch(error){
			return null;
		}
	}
}
```

아래는 AdCampaignAPI를 사용해서 여러 상품의 광고를 받아오는 로직이다. Promise.all을 사용해서 각 shop의 광고를 받아오고 있다.

```ts
const shopList = [
	{shopNo: 100, category: 'chicken'},
	{shopNo: 101, category: 'pizza'},
	{shopNo: 102, category 'noodle'}
];

const shopAdCampaignList = await Pormise.all(
	shopList.map((shop)=> AdCampaignAPI.operating(shop.shopNo))
);
```

이때 AdCampaignAPI.operating함수에서 null을 반환할 수 있기 때문에 shopAdCampaignList 타입은 Array<AdCampaignAPI[] | null>로 추론된다.

shopAdCampaignList 변수를 NonNullable 함수로 필터링하지 않으면 shopAdCampaignList를 순회할 때마다 고차 함수 내 콜백 함수에서 if문을 사용한 타입 가드를 반복하게 된다.

다음과 같이 NonNullable 함수를 사용하여 shopAdCampaignList를 필터링하면 null이 아닌 값만 추출할 수 있다.

```ts
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

### 5.4 불변 객체 타입으로 활용하기
컴포넌트나 함수에서 객체를 사용할 때 열린 타입으로 설정할 수 있다. 함수 인자로 키를 받아서 value를 반환하는 함수를 보자. 

키 타입을 해당 객체에 존재하는 키값으로 설정하는 것이 아니라 string으로 설정하면 getColorHex함수의 반환 값은 any가 된다.

colors에 어떤 값이 추가될지 모르기 때문이다.

```ts
const colors = {
	red: '#F45452',
	green: '#0c9252a'
	blue: '#1A7cff'
}

const getColorHex = (key: string) => colors[key];
```

이런 문제를 해결하기 위해 불변 객체 타입을 사용할 수 있다. 불변 객체 타입은 객체의 키값을 타입으로 설정할 수 있어 타입 안정성을 높일 수 있다.

```ts
const colors = {
    red: '#F45452',
    green: '#0c9252a'
    blue: '#1A7cff'
} as const;

type Color = keyof typeof colors;

const getColorHex = (key: Color) => colors[key];
```

1. Atom 컴포넌트에서 theme style 객체 활용하기

atom 단위의 작은 컴포넌트는 폰트 크기, 폰트 색상, 배경 색상 등 다양한 환경에서 유연하게 사용될 수 있도록 구현되어야 하는데 이러한 설정값은 props로 넘겨주도록 설계한다. 

하지만 props로 직접 값을 넘기게 될 경우 사용자가 모든 값을 인지해야하고 변경사항에 취향한 상태가 된다. 

이런 문제를 해결하기 위해 대부분의 프로젝트에서는 대부분의 프로젝트에서는 해당 프로젝트의 스타일 값을 관리해주는 theme 객체를 두고 관리한다. 

이때 theme 객체를 불변 객체 타입으로 설정하여 타입 안정성을 높일 수 있다.

```ts
const colors = {
	red: '#F45452',
	green: '#0C952A',
	blue: '#1A7CFF'
}

type ColorsType = typeof colors;
/**
{
	red: string;
	green: string;
	blue: string;
}
*/

import {FC} from 'react';
import styled from 'styled-components';
const colors = {
	black:'#000000',
	gray: '#222222',
	white: '#ffffff',
	mint: '#2ac1bc'
}

const theme = {
	colors: {
		default: colors.gray,
		...colors
	},
	backgdroundColor: {
		default: colors.white,
		gray: colors.gray,
		mint: colors.mint,
		black: colors.black
	},
	fontSize: {
		default: '16px',
		small: '14px',
		large: '18px'
	}
}

type ColorType = typeof keyof theme.colors;
type BackgroundColorType = typeof keyof theme.backgroundColor;
type FontSizeType = typeof keyof theme.fontSize;

interface Props {
	color?:ColorType;
	backgroundColor?: BackgroundColorType;
	fontSize?:FontSizeType;
	onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}

const Button: FC<Props> = ({fontSize, backgroundColor, color, children, onClick}) => {
	return (
		<ButtonWrap
			fontSize={fontSize}
			backgroundColor={backgroundColor}
			color={color}
		>
		{children}
		</ButtonWrap>
	)
}

const ButtonWrap = styled.button<Omit<Props, 'onClick'>>`
	color: ${({color}) => theme.color[color ?? 'default']};
	backgorund-color: ${({backgroundColor}) => theme.bgColor[backgroundColor ?? 'default']};
	font-size: ${({fontSize}) => theme.fontSize[fontSize ?? 'defulat']};
`
```

### 5.5 Record 원시 타입 키 개선하기
- 객체 선언 시 키가 어떤 값이 명확하지 않을 때 string이나 Number로 표기하는데 이는 예상치못한 런타임 에러를 야기할 수 있다.
- Record를 사용하면 객체의 키와 값의 타입을 명확하게 지정할 수 있다.

```ts
type Category = string;
interface Food {
	name: string;
	// ...
}

const foodByCategory: Record<Category, Food[]> = {
	한식: [{name:'제육덮밥'}, {name: '뚝배기 불고기'}],
	일식: [{name: '초밥'}, {name: '텐동'}]
}

foodByCategory['양식'];
foodByCategory['양식'].map((food) => console.log(food.name)); // 오류가 발생하지 않는다.
```

여기에서 Category의 타입은 string이다. foodByCategory 객체는 무한한 키 집합을 가지게 된다.
foodByCategory 객체에 없는 키 값을 사용하더라도 타입스크립트는 오류를 표시하지 않는다.

그러나 foodByCategory['양식']은 런타임에서 undefined가 되어 오류를 반환한다.

이때 자바스크립트의 옵셔널 체이닝 등을 사용해 런타임 에러를 방지할 수 있다. 하지만 어떤 값이 undefined인지 매번 판단해야 한다는 번거로움이 생긴다.

또한 실수로 옵셔널 체이닝을 놓치는 경우에 런타임 에러가 발생할 수 있다. 

이런 문제를 해결하기 위해 Record를 사용하여 객체의 키와 값의 타입을 명확하게 지정할 수 있다.

1. 유닛 타입으로 변경하기

```ts
type Category = '한식' | '일식';

interface Food {
	name: string;
	//...
}

const foodByCategory: Record<Category, Food[]> = {
	한식: [{name:'제육덮밥'}, {name: '뚝배기 불고기'}],
	일식: [{name: '초밥'}, {name: '텐동'}]
}
```

하지만 키가 무한해야 하는 상황에는 적합하지 않다.

2. Partial을 활용하여 정확한 타입 표현하기
```ts
type PartialRecord<K extends string, T> = Partial<Record<K, T>>;
type Category= string;

interface Food {
	name: string;
	// ...
}

const foodByCategory: PartialRecord<Category, Food[]> = {
	한식: [{name:'제육덮밥'}, {name: '뚝배기 불고기'}],
	일식: [{name: '초밥'}, {name: '텐동'}]
}

foodByCategory['양식'];
foodByCategory['양식'].map((food) => console.log(food.name)); // Object is possibly 'undefined'
```

PartialRecord<Category, Food[]>는 Category로 정의된 문자열 키들이 선택적으로 존재할 수 있다는 것을 나타낸다.
그렇기 때문에 foodByCategory['양식']은 undefined일 가능성이 있다고 타입스크립트가 경고해주고 이를 통해 안전하게 처리할 수 있다.


