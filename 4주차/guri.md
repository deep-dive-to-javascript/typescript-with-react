# 1. 조건부 타입

- 타입스크립트에서는  조건부 타입을 사용해 조건에 따라 출력 타입을 다르게 도출할 수 있음

## 1.1 extends와 제네릭을 활용한 조건부 타입

- 타입을 확장할떄, 타입을 조건부로 설정할 때 사용
- 제네릭타입에서 한정자 역할로 사용.

```jsx
T extends U ? X : Y
```

- 타입 T를 U에 할당할 수 있으면 X타입, 아니면 Y타입으로 결정됨.

```tsx
interface Bank {
	financialCode: string;
	companyName: string;
	name: string;
	fullName: string;
}

interface Card {
	financialCode: string;
	companyName: stirng;
	name: string;
	appCardType?: string;
}

type PayMethod<T> = T extends "card" ? Card : Bank;
type CardPayMethodType = PayMethod<"card">;
type BankPayMethodType = PayMethod<"bank">;
```

- 제네릭 매개변수에 “card”가 들어오면 Card 타입, 그 외의 값이 들어오면 Bank타입으로 결정된다.
- PayMethod를 사용해서 CardPayMethodType, BankPayMethodType을 도출할 수 있음.

## 1.2 조건부 타입을 사용하지 않을때의 문제점

```tsx
계좌 정보 엔드포인트: www.baemin.com/baeminpay/.../bank
카드 정보 엔드포인트: www.baemin.com/baeminpay/.../card
앱 카드 정보 엔드포인트: www.baemin.com/baeminpay/.../appcard
```

- 각 API는 계좌, 카드, 앱카드의 결제 수단 정보를 배열로 반환함
- 엔드포인트가 비슷하므로 서버 응답을 처리하는 공통 함수를 만들고, 함수에 타입을 전달하여 타입별로 로직 구현
- 사용되는 타입 :

```tsx
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

//최종적인 은행, 카드 결제 수단 타입이다. 프론트에서 추가되는 UI 데이터 타입과 제네릭으로 받아오는 Bank 또는 Card를 합성한다.
//extends를 제네릭에서 한정자로 사용하여 Bank또는 Card를 포함하지 않는 타입은 제네릭으로 넘겨주지 못하게 방어한다.
//BankPayMethodInfo = PayMethodInterface & Bank처럼 카드와 은행의 타입을 만들어줄 수 있지만 제네릭을 활용해서 중복된 코드를 제거한다.
type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;

//프론트에서 관리하는 결제 수단 관련 데이터로 UI를 구현하는 데 사용되는 타입이다.
type PayMethodInterface = {
	companyName: string;
	// ...
}
```

- react-query의 useQuery를 사용하여 구현한 커스텀 훅인 useGetRegisteredList함수

```tsx
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;
// useGetRegisteredList 함수는 타입으로 card, appcard, bank 를 받아 해당 결제소단 정보 리스트를 반환하는 함수임.useQuery의 반환값을 돌려준다
export const useGetRegisteredList = (
type: 'card' | 'appcard' | 'bank'
): useQueryResult<PayMethodType[]> => {
const url = `baeminpay/code/${type === 'appcard' ? 'card' : type}`;

    // fetcherFactory는 axios를 래핑해주는 함수이며, 서버에서 데이터를 받아온 후 onSuccess 콜백 함수를 거친 결과값을 반환한다
	const fetcher = fetcherFactory<PayMethodType[]>({
		onSuccess: (res) => {
			const usablePocketList = res?.filter(
				(pocket: PocketInfo<Card> | PocketInfo<Bank>) => pocket?.useType === 'USE'
			) ?? [];
			
			return usablePockerList;
		}
	});
    
	const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
	return result;
}
```

- useGetRegisteredList함수는 타입을 구분해서 넣는 사용자 의도와 다르게 정확한 타입을 반환하지 못하는 함수가 됐다.
- 타입 설정이 유니온으로만 되어있기 때문에 타입스크립트는 해당 타입에 맞는 Data 타입을 추론할 수 없다.
- 인자에 따라 반환되는 타입을 다르게 설정하고 싶다면 extends를 사용한 조건부 타입을 활용하면 된다.

## 1.3 extends 조건부 타입을 활용하여 개선하기

- useGetRegisteredList 함수의 반환 Data는 인자 타입에 따라 정해져 있다.
- 타입으로 ‘card’ 또는 ‘appcard’를 받으면 카드 결제 수단 정보 타입인 PocketInfo<card>를 반환하고, ‘bank’를 받는다면 PocketInfo<bank>를 반환한다.
    - type “card” | “appcard” ⇒ PocketInfo<Card>
    - type “bank” ⇒ PocketInfo<Bank>
- 조건부 타입을 사용해 PayMethodInfo<Card> | PayMethodInfo<Bank> 타입이었던 PayMethodType 타입을 개선해보자

```tsx
type PayMethodType<T extends "card" | "appcard" | "bank"> = T extends
| "card"
| "appcard" 
? Card 
: Bank;
```

- PayMethodType의 제네릭으로 받은 값이 “card”또는 “appcard”일때 PayMethodInfo<Card>타입, 아닐떄는 PayMethodInfo<Bank> 타입을 반환.
- 결제수단 타입에는 card, appcard, bank만 들어올 수 있기 떄문에 extends를 한정자로 활용해 제네릭에 넘겨오는 값을 제한함.
- 새롭게 정의한 PayMethod타입에 제네릭 값을 넣어주기 위해서는 useGetRegisteredList함수의 인자 타입을 넣어줘야함.
- useGetRegisteredList 인자 타입을 제네릭으로 받으면서 extends를 활용하여 card, appcard, bank이외의 다른 값이 인자로 들어오는 경우 타입에러를 반환

```tsx
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

- 조건부 타입을 활용하여 PayMethodType이 사용자가 인자에 넣는 타입 값에 맞는 타입만을 반환하도록 구현했다.
- 이로써 사용자는 useGetRegisteredList 함수를 사용할 때 불필요한 타입 가드를 하지 않아도 된다.
- PocketInfo<Card>만을 받는 컴포넌트의 props로 넘겨줄 때 불필요한 단언을 하지 않아도 됨.
- extends 활용 예시 정리
    - 제네릭과 extends를 함께 사용해 제네릭으로 받는 타입을 제한함. 휴먼에러를 방지할 수 있음.
    - extends를 활용해 조건부 타입을 설정. 반환 값을 사용자가 원하는 값으로 구체화하여 불필요한 타입 가드, 타입 단언을 방지할 수 있음.

## 1.4 infer를 활용해서 타입 추론하기

- infer는 조건부 타입에서 사용되는 키워드로, 타입을 추론할 때 사용된다.
- extends로 조건을 서술하고 interfer로 타입을 추론하는 방식.
- infer를 활용한 예시

```tsx
type UnpackPromise<T> = T extends Promise<infer K> ? K : any;
```

- UnpackPromise는 T가 Promise로 래핑된 경우라면 K를 반환하고 아니라면 any를 반환한다.
- Promise<interfer K>는 Promise의 반환 값을 추론해 해당 값의 타입을 K로 한다는 의미이다.

```tsx
const promises = [Promise.resolve('Hello'), Promise.resolve(123)];
type expected = UnpackPromise<typeof promises>; // string | number
```

- 배민 라이더를 관리하는 어드민 서비스예시 : 사용하는 타입

```tsx
//라이더 어드민에서 라우팅을 위해 사용하는 타입.
interface RouteBase {
	name: string;
	path: string;
	component: ComponentType;
}
//RouteItem의 name은 pages가 있을 때는 단순히 이름의 역할만 하며 그렇지 않을 때는 사용자 권한과 비교한다.
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

```tsx
//메뉴 리스트에서 사용하는 타입으로 권한 API를 통해 반환된 사용자 권한과 name을 비교하여 사용자가 접근할 수 있는 메뉴만 렌더링한다.
export interface SubMenu {
	name: string;
	path: string;
}
//MainMenu의 name은 subMenus를 가지고 있을 때 단순히 이름의 역할만 하며, 그렇지 않을 때는 권한으로 간주된다.
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

- menuList에는 MainMenu와 subMenu타입이 올 수 있기때문에 이를 유니온 타입으로 정의한다. 따라서 menuList에서 subMenus가 없는 MainMenu의 name과 subMenus에서 쓰의는 name, route name에 동일한 문자열만 입력해야한다는 제약이 존재함.
- 하지만 name은 string 타입으로 정의되어 있기 때문에 routes와 menuList에서 subMenu의 기기 내역 관리처럼 서로 다른 값이 입력되어도 컴파일타임에서 에러가 발생하지 않는다. 존재하지 않는 권한에 대한 문제로 잘못 인지할 수 있음.

```tsx
type PermissionNames = '기기 정보 관리' | '안전모 인증 관리' | '운행 여부 조회'; //...
```

- 이를 개선하기 위해 PermissionNames 처럼 별도 타입을 선언하여 name을 관리하는 방법도 있지만, 권한 검사가 필요 없는 subMenu나 pages가 존재하는 name을 따로 처리해야한다.
- 이때 infer와 불변 객체(as const)를 활용해서 menuList 또는 routes의 값을 추출하여 타입으로 정의하는 식으로 개선할 수 있다.

```tsx
//subMenu의 타입을 ReadonlyArray로 변경
export interface MainMenu {
	// ...
	subMenu?: ReadonlyArray<SubMenu>;
}

// as const키워드를 추가하여 불변객체로 정의
export const menuList = [
	// ...
] as const;

//Route 관련 타입의 name은 menuList의 name에서 추출한 타입인PermissionNames만 올 수 있도록 변경
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

```tsx
//조건에 맞는 값을 추출한 UnpackMenuNames라는 타입 추가
type UnpackMenuNames<T extends ReadonlyArray<MenuItem>> = T extends ReadonlyArray<infer U>
	? U extedns MainMenu
		? U['subMenus'] extends infer V
			? V extends ReadonlyArray<SubMenu> ? UnpackMenuNames<V> : U['name']
			: never
		: U extends SubMenu ? U['name'] : never
	:never;

export type PermissionNames = UnpackMenuNames<typeof menuList>; // PermissionNames = '기기 내역 관리' | '헬멧 인증 관리'
```

- U가 MainMenu타입이라면 subMenus를 infer V로 추출한다.
- subMenus는 옵셔널한 타입이기 때문에 추출한 V가 존재한다면(SubMenu 타입에 할당할 수 있다면) UnpackMenuNames에 다시 전달한다.
- V가 존재하지 않는다면 MainMenu의 name은 권한에 해당하므로 U['name']이다.
- U가 MainMenu가 아니라 SubMenu에 할당할 수 있다면(U는 SubMenu 타입이기 때문에) U['name']은 권한에 해당한다.

```tsx
export type PermissionNames = UnpackMenuNames<typeof menuList>;
//[기기내역관리, 헬멧인증관리, 운행관리]
```

- PermissionNames는 menuList에서 권한으로 유요한 값만 추출하여 배열로 반환하는 타입임을 확인할 수 있다.

# 2. 템플릿 리터럴 타입 활용하기

- 타입스크립트에서는 유니온 타입을 사용하여 변수 타입을 특정 문자열로 지정할 수 있음

```tsx
type HeaderTag = "h1" | "h2" | "h3" | "h4" | "h5";
```

- 컴파일타임의 변수에 할당되는 타입을 특정 문자열로 정확하게 검사하여 휴먼 에러를 방지, 자동완성 기능을 통해 개발 생산성을 높일 수 있음.
- 타입스크립트 4.1부터 이를 확장하는 방법인 템플릿 리터럴 타입을 지원하기 시작

```tsx
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;
```

- 수평 또는 수직 방향을 표현하는 Direction타입을 다음과 같이 표현할 수 있음

```tsx
//유니온타입
type Direction = 
| "top"
| "topLeft"
| "topRight"
| "bottom"
| "bottomLeft"
| "bottomRight";

//템플릿 리터럴 타입
type Vertical = 'top' | 'bottom';
type Horizon = 'left' | 'right';

type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}}`;
```

- 템플릿 리터럴 타입을 사용하면 가독성이 좋고 재사용과 수정이 용이한 타입을 선언할 수 있음.
- 타입스크립트 컴파일러가 유니온을 추론하는데 시간이 오래 걸리면 비효율적이기 때문에 타입스크립트가 타입을 추론하지않고 에러를 뱉는 경우가 있음. 따라서 유니온 조합의 경우의 수가 너무 많지 않게 적절히 나누어 조합할 필요가 있다.

# 3. 커스텀 유틸리티 타입 활용하기

- 타입스크립트에서 제공하는 유틸리티 타입만으로 표현하는데 한계를 느끼는 경우 유틸리티 타입을 활용하여 커스텀 유틸리티 타입을 제작하여 사용함.

## 3.1 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

- props에서 받은 타입을 styled-components로 넘겨주는 경우 Pick, Omit같은 유틸리티 타입을 활용하여 코드를 간결하게 작성할 수 있다.
- 유틸리티  타입을 활용하지 않는경우
    - Props타입과 styled-components 타입의 중복선언 및 문제점
        - 수평선을 그어주는 Hr 컴포넌트 예시
        
        ```tsx
        //HrComponent.tsx
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
        
        - Hr 컴포넌트는 className, height, color, isFull 속성을 Props로 받음
        - 이 속성은 styled-components인 HrComponent에도 바로 연결되며 타입도 같다.
        - styledProps를 따로 정의하려면 Props와 똑같은 타입임에도 새로 작성해야 하므로 불가피하게 중복된 코드가 생긴다. 그리고 Props가 변경되면 styledProps도 함께 변경되어야한다.
        - 이런 문제를 Pick, Omit과 같은 유틸리티 타입으로 개선할 수 있다.
        
        ```tsx
        type StyledProps = Pick<Props, "height" | "color" | "isFull">;
        ```
        
        - Pick 유틸리티 타입을 활용, props에서 필요한 부분만 선택하여 styled-components 컴포넌트의 타입을 정의하면, 중복된 코드를 작성하지 않아도 되고 유지보수를 편리하게 할 수 있다.

## 3.2 PickOne 유틸리티 함수

- 타입스크립트에서는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 있음
- 이를위해 PickOne이라는 이름의 유틸리티 함수를 구현

```tsx
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

- Card, Account중 하나의 객체만 받고 싶은 상황에서 Card | Account로 타입을 작성하면 의도한 대로 타입 검사가 이뤄지지않음.
- 또한 withdraw의 인자로 {card: 'hyundai'} {account: 'hana'} 중 하나만 받고 싶지만 실제로는 둘 다 받아도 타입 에러가 일어나지 않음.
- 집합 관점으로 볼 때 유니온은 합집합임. card, account 속성이 하나씩만 할당된 상태와 card, account속성이 모두 포함된 상태 둘 다 합집합의 범주에 들어가기 때문에 타입 에러가 발생하지 않는다.
- 이때 식별할 수 있는 유니온 기법을 자주 활용한다.

**식별할 수 있는 유니온으로 객체 타입을 유니온으로 받기**

- 각 타입에 type이라는 공통된 속성을 추가하여 구분짓는 방법.

```tsx
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

- 일일히 type을 다 넣어줘야하는 불편함이 생김. 이미 구현된 상태에서 식별할 수 있는 유니온을 적용하려면 해당 함수를 사용하는 부분을 모두 수정해야함.
- 이 상황을 방지하기 위해 PickOne 유틸리티 타입을 구현하여 적용해보자

**PickOne 커스텀 유틸리티 타입 구현하기**

- account일때는 card를 받지 못하고, card일때는 account를 받지 못하게 하려면 하나의 속성이 들어왔을 때 다른 타입을 옵셔널한 undefined로 만들어주면 된다.
- 옵셔널 + undefined로 타입을 지정하면 사용자가 의도적으로 undefined에 값을 넣지 않는 이상, 원치 않는 속성에 값을 넣었을 때 타입 에러가 발생할것.

```tsx
{account: string; card?: undefined} | {account?: undefined; card: string} 
```

- 이를 커스텀 유틸리티 타입으로 구현하면 다음과 같다.

```tsx
type PickOne<T> = {
	[P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];
```

**PickOne 살펴보기**

- One<T>

```tsx
type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];
```

1. [P in keyof T]에서 T는 객체로 가정하기 때문에 P는 T 객체의 키 값을 말한다.
2. Record<P, T[P]>는 P타입을 키로 가지고, value는 P를 키로 둔 T 객체의 값의 레코드 타입을 말한다.
3. 따라서{ [P in keyof T]: Record<P, T[P]> }에서 키는 T 객체의 키 모음이고, value는 해당 키의 원본 객체 T를 말한다.
4. 3번의 타입에서 다시 [keyof T]의 키 값으로 접근하기 때문에 최종 결과는 전달받은 T와 같다.

```tsx
type Card = { card: string };
const one: One<Card> = { card: "hyundai" };
```

- ExcludeOne<T>

```tsx
type ExcludeOne<T> = { [P in keyof T]: Partial<Record<Exclude<keyof T, P>, undefined>> }[keyof T];
```

1. [P in keyof T]에서 T는 객체로 가정하기 때문에 P는 T 객체의 키값을 말한다.
2. Exclude<keyof T, P>는 T 객체가 가진 키값에서 P 타입과 일치하는 키값을 제외한다. 이 타입을 A라고 하자.
3. Record<A, undefined>는 키로 A 타입을, 값으로 undefined 타입을 갖는 레코드 타입이다. 즉, 전달 받은 객체 타입을 모두 { [key]: undefined } 형태로 만든다. 이 타입을 B라고 하자.
4. Partial<B>는 B 타입을 옵셔널로 만든다. 따라서 { [key]?: undefined }와 같다.
5. 최종적으로 [P in keyof T]로 매핑된 타입에서 동일한 객체의 키값인 [keyof T]로 접근하기 때문에 4번 타입이 반환된다.

앞의 속성을 활용해 PickOne타입을 표현할 수 있음.

- PickOne<T>

```tsx
type PickOne<T> = One<T> & ExcludeOne<T>;
```

1. One<T> & Exclude<T>는 [P in keyof T]를 공통적으로 갖기 떄문에 아래와 같이 교차된다.

```tsx
[P in keyof T]: Record<P, T[P]> & Parial<Record<Exclude<keyof T, P>, undefined>>
```

1. 이 타입을 해석하면 전달된 T 타입의 1개의 키는 값을 가지고 있으며, 나머지 키는 옵셔널한 undefined값을 가진 객체를 의미한다.

**PinkOne 타입 적용하기**

```tsx
type Card = {
    card: string;
}

type Account = {
    account: string;
}

type CardOrAccount = PickOne<Card & Account>;
function withdraw(type: CardOrAccount){
    ...
}

withdraw({ card: 'hyundai', account: 'haha' }); // 에러 발생
```

- 커스텀 유틸리티 타입을 구현할 땐 정확히 어떤 타입을 구현해야하는지 파악하고 필요한 타입을 작은 단위로 쪼개어 생각하여 단계적으로 구현하는 것이 좋다.

## 3.3 NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

- 일반적으로 if문을 이용해서 null 처리 타입가드를 하지만 is와 NonNullable타입으로 유틸 함수를 만들어 사용할 수 있다.

**NonNullable 타입이란**

- 타입스크립트에서 제공하는 유틸리티 타입. 제네릭으로 받는 T가 null 또는 undefined일때 never 또는 T를 반환하는 타입.
- NonNullable을 사용하면 null이나 indefined가 아닌 경우를 제외할 수 있다.

```tsx
type NonNullable<T> = T extends null | undefined ? never : T;
```

**null, undefined를 검사해주는 NonNullable 함수**

- NonNullable 유틸리티 타입을 사용하여 null 또는 undefined를 검사해주는 타입 가드 함수를 만들어 쓸 수 있다.
- NonNullable 함수는 매개변수인 value가 null 또는 undefined라면 false를 반환한다.
- is 키워드가 쓰였기 때문에 NonNullable 함수를 사용하는 쪽에서 true가 반환된다면 넘겨준 인자는 null이나 undefined가 아닌 타입으로 타입 가드가 된다.

```tsx
function NonNullable<T>(value: T) = value is NonNullable<T> {
	return value !== null && value !== undefined;
}
```

**Promise.all을 사용할 때 NonNullable 적용하기**

- 상품 번호인 shopNumber 매개변수에 따라 각기 다른 응답 값을 반환하는 광고 조회 API
- 여러 상품 조회시 하나의 API에서 에러가 발생한다고 해서 전체 광고가 보이지 않으면 안됨. 따라서 try-catch문을 사용하여 에러가 발생할 때는 null을 반환하고 있음.

```tsx
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

- 아래는 AdCampaignAPI를 사용해서 여러 상품의 광고를 받아오는 로직이다. Promise.all을 사용해서 각 shop의 광고를 받아오고 있다.

```tsx
const shopList = [
	{shopNo: 100, category: 'chicken'},
	{shopNo: 101, category: 'pizza'},
	{shopNo: 102, category 'noodle'}
];

const shopAdCampaignList = await Pormise.all(
	shopList.map((shop)=> AdCampaignAPI.operating(shop.shopNo))
);
```

- 이때 AdCampaignAPI.operating함수에서 null을 반환할 수 있기 때문에 shopAdCampaignList 타입은 Array<AdCampaignAPI[] | null>로 추론된다.
- shopAdCampaignList 변수를 NonNullable 함수로 필터링하지 않으면 shopAdCampaignList를 순회할 때마다 고차 함수 내 콜백 함수에서 if문을 사용한 타입 가드를 반복하게 된다.
- 다음과 같이 NonNullable 함수를 사용하여 shopAdCampaignList를 필터링하면 원하는 타입인 Array<AdCampaignAPI[] >로 추론할 수 있게 된다.

```tsx
...

const shopAds = shopAdCampaignList.filter(NonNullable);
```

# 4. **불변 객체 타입으로 활용하기**

- 컴포넌트나 함수에서 객체를 사용할 때 열린 타입으로 설정할 수 있다. 함수 인자로 키를 받아서 value를 반환하는 함수를 보자.
- 키 타입을 해당 객체에 존재하는 키값으로 설정하는 것이 아니라 string으로 설정하면 getColorHex함수의 반환 값은 any가 된다. colors에 어떤 값이 추가될지 모르기 때문이다.

```tsx
const colors = {
	red: '#F45452',
	green: '#0c9252a'
	blue: '#1A7cff'
}

const getColorHex = (key: string) => colors[key];
```

- as const 키워드로 객체를 불변 객체로 선언하고, keyof 연산자를 사용하여 getColorHex 함수 인자로 실제 colors 객체에 존재하는 키값만 받도록 설정할 수 있다.
- 객체 타입을 구체적으로 설정하면 타입에 맞지 않는 값을 전달할 경우 타입 에러가 반환되어 컴파일 단계에서 발생할 수 있는 실수를 방지 할 수 있다.
- 또한 자동완성 기능으로 객체에 어떤 값이 있는지 쉽게 파악할 수 있다.

## 4.1 Atom 컴포넌트에서 theme style 객체 활용하기

- Atom 단위의 작은 컴포넌트는 폰트 크기, 폰트 색상, 배경 색상 등 다양한 환경에서 유연하게 사용될 수 있도록 구현되어야 하는데 이러한 설정값은 props로 넘겨주도록 설계한다.
- props로 직접 색상 값을 넘겨주면 사용자가 모든 값을 인지해야하고 변경에 취약한 상태가 된다.
- 대부분 프로젝트에서는 해당 프로젝트의 스타일 값을 관리하는 theme 객체를 두고 관리함.
- props에 잘못된 값을 넘겨주는걸 방지하고 자동완성이 되도록 하기 위해 theme 객체로 타입을 구체화하여 사용한다.

**타입스크립트 keyof 연산자로 객체의 키값을 타입으로 추출**

- keyof 연산자는 객체 타입을 받아 해당 객체의 키값을 string 또는 number의 리터럴 유니온 타입을 반환한다.

```tsx
interface ColorType {
	red: string;
	green: string;
	blue: string;
}

type ColorKeyType = Keyof ColorType;//'red' | 'green' | 'blue'
```

**타입스크립트 typeof 연산자로 값을 타입으로 다루기**

- keyof 연산자는 객체 타입을 받는다. 따라서 객체의 키값을 타입으로 다루려면 값 객체를 타입으로 변환해야함. 이때 typeof 사용.
- 변수 혹은 속성의 타입을 추론하는 역할

```tsx
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
```

객체의 타입을 활용해서 컴포넌트 구현하기

```tsx
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
	...
```

# 5. Record 원시 타입 키 개선하기

- 객체 선언 시 키가 어떤 값인지 명확하지 않다면 Record 의 키를 string이나 number같은 원시 타입으로 명시하곤 한다.
- 이떄 타입스크립트는 키가 유효하지 않더라도 타입상으로는 문제가 없기 때문에 오류를 표시하지 않는데 이는 예상치 못한 런타임 에러를 야기할 수 있음.
- Record를 명시적으로 사용하는 방안에 대해 알아봄

## 5.1 무한한 키를 집합으로 가지는 Record

```tsx
type Category = string;
interface Food {
	name: string;
	// ...
}

const foodByCategory: Record<Category, Food[]> = {
	한식: [{name:'제육덮밥'}, {name: '뚝배기 불고기'}],
	일식: [{name: '초밥'}, {name: '텐동'}]
}
```

- 여기에서 Category의 타입은 string이다. foodByCategory 객체는 무한한 키 집합을 가지게 된다.
- foodByCategory 객체에 없는 키 값을 사용하더라도 타입스크립트는 오류를 표시하지 않는다.

```tsx
foodByCategory['양식'];
foodByCategory['양식'].map((food) => console.log(food.name)); // 오류가 발생하지 않는다.
```

- 그러나 foodByCategory['양식']은 런타임에서 undefined가 되어 오류를 반환한다.
- 자바스크립트의 옵셔널 체이닝 등을 사용해 런타임 에러를 방지할 수 있다.

```tsx
foodByCategory['양식']?.map((food) => console.log(food.name));
```

- 어떤 값이 undefined인지 매번 판단해야 한다는 번거로움이 생긴다. 실수로 undefined일 수 있는 값을 인지하지 못하고 코드를 작성하면 예상치 못한 런타임 에러가 발생할 수 있다.

## 5.2 유닛 타입으로 변경하기

- 키가 유한한 집합이면 유닛 타입을 사용할 수 있다.

```tsx
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

- 이제 foodByCategory에 없는 값을 사용하면 에러가 발생한다. 유닛타입으로 개발중에 유효한 키가 사용되었는지 확인할 수 있다.
- 하지만 키가 무한해야하는 상황에는 적합하지 않다.

## 5.3 Partial을 활용하여 정확한 타입 표현하기

- 객체 값이 undefined일수 있는 경우 Partial을 사용해서 PartialRecord 타입을 선언하고 객체를 선언할때 이를 활용

```tsx
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
