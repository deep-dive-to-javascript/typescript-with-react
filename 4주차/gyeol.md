# 5장 타입 활용하기

## 5.1 조건부 타입

- 타입스크립트의 조건부 타입은 자바스크립트의 삼항 연산자와 동일한 형태를 가짐
- `Condition ? A : B`
    - A는 Condition이 true일 때 도출되는 타입,  B는 false일 때 도출되는 타입

### 1. extends와 제네릭을 활용한 조건부 타입

- `T extends U ? X : Y`
    - 타입 T를 U에 할당할 수 있으면 X 타입, 아니면 Y 타입으로 결정
    
    ```tsx
    // 제네릭이 string 이면 문자열 배열, 아니면 넘버 배열 
    type isStringType<T> = T extends string ? string[] : number[];
    
    type T1 = isStringType<string>
    type T2 = isStringType<number>
    
    const a: T1 = ['가','나','다'];
    const b: T2 = [1,2,3];
    
    // 제네릭 `T`는 `boolean` 타입으로 제한.
    // 제네릭 T에 true가 들어오면 string 타입으로, 
    // false가 들어오면 number 타입으로 data 속성을 타입 지정
    interface isDataString<T extends boolean> {
       data: T extends true ? string : number;
       isString: T;
    }
    
    const str: isDataString<true> = {
       data: '홍길동', // String
       isString: true,
    };
    
    const num: isDataString<false> = {
       data: 9999, // Number
       isString: false,
    };
    ```
    

### 2. 조건부 타입을 사용하지 않았을 때의 문제점

📎 **코드로 살펴보기**

- 구현할 코드 설명
    - 계좌, 카드, 앱카드 등 3가지 결제 수단 정보를 가져오는 API가 존재하며 결제 수단 정보를 배열 형태로 반환함
    - 세 가지 API의 엔드포인트가 비슷하기 때문에 서버 응답을 처리하는 공통 함수를 생성하고, 해당 함수에 타입을 전달하여 타입 별로 처리로직 구현
- 사용되는 타입 정의
    
    ```tsx
    interface PayMethodBaseFromRes {
      financialCode: string;
      name: string;
    }
    interface Bank extends PayMethodBaseFromRes {
      fullName: string;
    }
    interface Card extends PayMethodBaseFromRes {
      appCardType?: string;
    }
    type PayMethodInterface = {
      companyName: string;
      // ...
    }
    type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;
    ```
    
    - `PayMethodBaseFromRes`
        - 서버에서 받아오는 결제 수단 기본타입
        - 은행과 카드에 모두 들어가 있음
    - `Bank, Card`
        - 은행과 카드 각각에 맞는 결제수단 타입
        - PayMethodBaseFromRes을 상속받아 구현
    - `PayMethodInterface`
        - 프론트에서 관리하는 결제수단 관련 데이터로 UI를 구현하는데 사용되는 타입
    - `PayMethodInfo<T extends Bank | Card>`
        - 최종적인 은행, 카드 결제수단 타입. 프론트에서 추가되는 UI 데이터 타입과 제네릭으로 받아오는 Bank 또는 Card를 합성함
        - extends를 제네릭에서 한정자로 사용 
        → Bank 또는 Card를 포함하지 않는 타입은 제네릭으로 넘겨주지 못하도록 방어함
        - PayMethodInfo = PayMethodInterface & Bank 처럼 카드와 은행의 타입을 만들어줄 수 있지만 제네릭을 활용해서 중복된 코드를 제거
- react-query의 `useQuery` 를 사용하여 구현한 커스텀 훅인 useGetRegisteredList 함수
    
    ```tsx
    type PayMethodType = PayMethodInfo<Bank> | PayMethodInfo<Card>
    export const useGetRegisteredList = (
    	type: "card" | "appcard" | "bank"
    ): UseQueryResult<PayMethodType[]> => {
      const url = `baeminpay/codes/${type ===  "appcard" ? "card" : type}`;
      
      const fetcher = fetcherFactory<PayMethodType[]>({
        onSuccess: (res) => {
          const usablePocketList = 
            res?.filter((pocket: PocketInto<Card> | PocketInto<Bank>) =>
              pocket?.useType === "USE"
            ) ?? [];
          return usablePocketList;
        },
      });
    
      const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
      return result;
    };
    ```
    
    - `useGetRegisteredList`
        - useQuery의 반환값을 리턴
        - 타입으로 "card" | "appcard" | "bank"를 받아서 해당 결제 수단의 결제 수단 정보 리스트를 반환
    - `useCommonQuery<T>`
        - useQuery를 한번 래핑해서 사용하고 있는 함수
        - useQuery의 반환 data를 T타입으로 반환함
    - `fetcherFactory`
        - axios를 래핑해주는 함수
        - 서버에서 데이터를 받아온 후 on Success 콜백 함수를 거친 결괏값을 반환함
- 문제 상황
    - `useGetRegisteredList` 함수가 반환하는 Data 타입은 **PocketInto<Card> | PocketInto<Bank>**
    - 사용자가 타입으로 “card”를 넣었을 때 반환되는 Data 타입은 **PocketInto라고 유추할 수 있음**
    - 하지만 해당 함수가 반환하는 Data타입은 **`PayMethodType`** 이기 때문에 사용하는 쪽에서는 PocketInfo일 가능성도 존재
    → 이거 PocketInfo 아니고 PayMethodInfo 아님..?ㅠ
        
        ⇒ 사용자가 인자로 “card”를 전달했을 때 함수가 반환하는 타입이  `PayMethodInfo<Card>[]` 라면 좋겠지만 타입 설정이 유니온으로만 되어있기 때문에 구체적인 추론이 불가능함
        
    - 즉 `useGetRegisteredList` 함수는 인자로 주어진 타입에 맞는 타입을 반환하지 못하는 함수임
    - 인자에 따라 반환되는 타입을 다르게 설정하고 싶다면 extends를 사용한 조건부 타입을 활용하면 됨

### 3. extends 조건부 타입을 활용하여 개선하기

- 조건부 타입을 활용하면 하나의 API 함수에서 타입에 따른 정확한 반환 타입을 추론하게 만들 수 있음
- `extends` 를 한정자로 활용해서 제네릭에 넘겨오는 값을 제한하여 결제 수단타입으로 "card", "appcard", "bank” 외의 값이 들어오는 경우를 방어
    
    ```tsx
    // 개선 전
    type PayMethodType = PayMethodInfo<Bank> | PayMethodInfo<Card>
    
    // 개선 후
    type PayMethodType<T extends "card" | "appcard" | "bank"> = T extends
    	| "card"
    	| "appcard"
    	? Card
    	: Bank;
    // PayMethodType의 제네릭으로 받은 값이 “card” 또는 “appcard”면 
    // PayMethodType<Card> 타입 반환
    // 그 외의 값이면 PayMethodType<Bank> 타입 반환
    ```
    
- 적용된 코드 확인
    
    ```tsx
    // 개선 전
    export const useGetRegisteredList = (
    	type: "card" | "appcard" | "bank"
    ): UseQueryResult<PayMethodType[]> => {
      // ...
      const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
      return result;
    };
    // 개선 후
    export const useGetRegisteredList = <T extends "card" | "appcard" | "bank">(
    	type: T
    ): UseQueryResult<PayMethodType[]> => {
      // ...
      const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
      return result;
    };
    ```
    
    - 조건부 타입을 활용하여 사용자가 인자에 넣는 타입 값에 맞는 타입만을 반환하도록 구현했음
        - 인자로 “card” 또는 “appcard”를 넣으면 `PayMethodType<Card>` 타입을 반환
        - 인자로 “bank”를 넣으면 `PayMethodType<Bank>` 타입을 반환
    - 이로써 사용자는 불필요한 타입 가드와 불필요한 타입 단언을 하지 않아도됨

📌 **extends 활용 예시 정리**

- 제네릭과 extends를 함께 사용해 제네릭으로 받는 타입을 제한
    - 개발자가 잘못된 값을 넘기는 휴먼 에러 방지
- extends를 활용해 조건부 타입을 설정
    - 반환 값을 사용자가 원하는 값으로 구체화함
        
        → 불필요한 타입 가드, 타입 단언 방지
        

### 4. infer를 활용해서 타입 추론하기

- extends로 조건을 서술하고 infer로 타입을 추론할 수 있음
    
    ```tsx
    type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;
    // UnpackPromise 타입은 제네릭으로 T를 받아 
    // T가 Promise로 래핑된 경우엔 K를 반환하고
    // 그렇지 않은 경우엔 any를 반환함
    ```
    
    - `Promise<infer K>`
        - Promise의 반환값을 추론해 해당 값의 타입을 K로 한다는 의미
    
    ```tsx
    const promises = [Promise.resolve("Mark"), Rromise.resolve(38)];
    type Expected = UnpackPromise<typeof promises>; // string | number
    ```
    
    - 이처럼 extends와 infer, 제네릭을 활용하면 타입을 조건에 따라 더 세밀하게 사용할 수 있음

## 5.2 템플릿 리터럴 타입 활용하기

- 타입스크립트에서는 유니온 타입을 통해 변수 타입을 특정 문자열로 지정이 가능
- 이를 통해 컴파일타임의 변수에 할당되는 타입을 특정 문자열로 정확하게 검사하여 휴먼 에러를 방지할 수 있고, 자동완성 기능을 통해 개발 생산성을 높일 수 있음
- 타입스크립트 4.1부터 이를 확장하는 방법인 `템플릿 리터럴 타입`을 지원
- 템플릿 리터럴 타입 예시
    
    ```tsx
    // before
    type HeaderTag = "h1" | "h2" | "h3" | "h4" | "h5";
    // after
    type HeadingNumber = 1 | 2 | 3 | 4 | 5l
    type HeaderTag = `h${HeadingNumber}`;
    ```
    
    ```tsx
    // before
    type Direction =
     | "top"
     | "topLeft"
     | "topRight"
     | "bottom"
     | "bottomLeft"
     | "bottomRight"
    // after
    type Vertical = "top" | "bottom";
    type Horizon = "left" | "right";
    type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}`;
    ```
    
- 템플릿 리터럴 타입의 장점
    - 가독성이 좋은 코드 작성이 가능
    - 코드를 재사용하고 수정하는데 용이한 타입 선언이 가능
- 템플릿 리터럴 타입 사용 시 주의점
    - 타입스크립트 컴파일러가 유니온을 추론하는 데 시간이 오래걸리면 타입을 추론하지 않고 에러를 뱉는 경우가 존재함(비효율적이기 때문에)
        
         → 유니온 조합의 경우의 수가 너무 많지 않게 적절히 나누어 타입을 정의할 것
        

## 5.4 커스텀 유틸리티 타입 활용하기

- 타입스크립트에서 제공하는 유틸리티 타입만으로는 표현하는 데 한계가 있는 경우 커스텀 유틸리티 타입을 제작해서 사용할 수 있음

### 1. 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

- 리액트 컴포넌트 구현시 background-color, size 값 등을 props로 받아와서 상황에 맞게 스타일링을 구현할 때가 많음
- 이와 같은 스타일 관련 props는 styled-components에 전달되며 styled-components에도 해당 타입을 정확하게 작성해야함
- 이런 경우 타입스크립트에서 제공하는 `Pick`, `Omit` 같은 유틸리티 타입을 활용할 수 있음
- Props타입과 styled-components 타입의 중복 선언 및 문제점
    
    ```tsx
    // HrComponent.tsx
    // HrComponent은 styled-componets임
    export type Props = {
    	height?: string;
    	color?: keyof typeof colors;
    	isFull?: boolean;
    	className?: string;
    	// ...
    }
    
    // Hr 컴포넌트의 Props 중 height, color, isFull은 
    // HrComponent에 바로 연결되며 타입도 같음
    export const Hr: VFC<Props> = ({ height, color, isFull, className}) => {
    	// ...
    	return (
    		<HrComponent 
    			height={height} 
    			color={color} 
    			isFull={isFull} 
    			className={className} 
    		/>
    	);
    }
    ```
    
    - height, color, isFull에 대한 StyledProps 타입을 새로 정의하여 HrComponent에 적용하려고 할 경우
        - Props와 똑같은 타입임에도 새로 작성해야함 → 중복된 코드가 생김
        - Props의 height, color, isFull 타입이 변경되면 StypedProps도 같이 변경되어야함
    
    🚨 컴포넌트가 커지고 styled-components로 만든 컴포넌트가 늘어날 수록 중복되는 타입이 많아지므로 개선이 필요함
    
    **🔧 `Pick` , `Omit` 같은 유틸리티 타입으로 개선할 수 있음**
    
- `Pick`  을 사용하여 개선하기
    
    ```tsx
    // style.ts
    import { Props } from '...';
    type StyledProps = Pick<Props, "height" | "color" | "isFull">;
    // Pick 유틸리티 타입을 활용해서 props에서 필요한 부분만 선택하여 사용 가능
    const HrComponent = styled.hr<StyledProps>`
      height: ${({ height }) = > height || "10px"};
      margin: 0;
      background-color: ${({ color }) = > colors[color || "gray7"]};
      border: none;
      ${({ isFull }) => 
    	  isFull && 
    	  css`
    	    margin: 0 -15px;
    	  `}
    `;
    ```
    
    - 이처럼 중복된 코드를 작성하지 않아도 되고 유지보수를 더욱 편하게 할 수 있음

### 2. PickOne 유틸리티 함수

- 타입스크립트에는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 존재함
    
    ```tsx
    type Card = {
    	card: string
    };
    type Account = {
    	account: string
    };
    function withdraw(type: Card | Account){
    	// ...
    }
    withdraw({ card: "hyundai", account: "hana" });
    // Card와 Account 속성을 한 번에 받았기 때문에
    // 타입에러를 예상했지만 타입 에러가 발생하지 않는다.
    ```
    
    - 집합 관점에서 볼 때 유니온 타입은 합집합이 됨
        
        → withdraw의 인자인 type은 card, account 속성이 모두 포함되어도 합집합의 범주에 들어가기 때문에 타입 에러가 발생하지 않는다.
        

**1️⃣ 해결 방법 - 식별할 수 있는 유니온 활용**

- 식별할 수 있는 유니온
    - 각 타입에 type이라는 공통된 속성을 추가하여 구분 짓는 방법
- 타입을 구분할 수 있는 판별자로 `type`이라는 속성을 추가하여 withdraw 함수를 사용하는 곳에서 정확한 타입을 추론할 수 있도록 함
    
    ```tsx
    type Card = {
    	type: "card";
    	card: string;
    };
    type Account = {
    	type: "account"
    	account: string;
    };
    function withdraw(type: Card | Account){
    	// ...
    }
    withdraw({ type: "card", card: "hyundai" });
    withdraw({ type: "account", card: "hana" });
    ```
    
- 이미 구현된 상태에서 적용하려면 해당 함수를 사용하는 부분을 모두 수정해야하므로 불편함

**2️⃣ 해결 방법 - PickOne 커스텀 유틸리티 타입 구현하기**

- 위와 같은 상황을 방지하기 위해 커스텀 유틸리티 타입을 만들 수 있음
- 예시에서 구현하고자 하는 타입은 account 또는 card 속성 하나만 존재하는 객체를 받는 타입
- account 타입일 때는 card를 받지 못하고, card일 때는 account를 받지 못하게 하려면 **하나의 속성이 들어왔을 때 다른 타입을 옵셔널한 undefined 값으로 지정하는 방법**이 필요함
    
    ```tsx
    {account: string; card?: undefined} | {account?: undefined; card: string}
    ```
    
    - 옵셔널 + undefined로 타입을 지정하면 사용자가 의도적으로 undefined 값을 넣지 않는 이상 원치 않는 속성에 값을 넣었을 때 타입에러가 발생할 것
- account, card, payMoney 속성 중 하나만 필수로 받는 PayMethod 구현하기
    
    ```tsx
    type PayMethod = 
    	| {account: string; card?: undefined; payMoney?: undefined}
    	| {account?: undefined; card: string; payMoney?: undefined}
    	| {account?: undefined; card?: undefined; payMoney: string};
    // 선택하고자 하는 하나의 속성을 제외한 나머지 값을 옵셔널 타입 + undefined로 설정
    ```
    
- 위 코드를 유틸리티 타입으로 구현하기
    
    ```tsx
    type PickOne<T> = {
      [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
    }[keyof T];
    ```
