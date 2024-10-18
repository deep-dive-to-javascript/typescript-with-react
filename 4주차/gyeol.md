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
    
**📌 PickOne 살펴보기**

- PickOne 타입을 `One<T>`과 `ExcludeOne<T>`로 분리해서 생각 할 수 있음
(T에는 객체가 들어 온다고 가정하고 진행)
    - `One<T>`
    
    ```tsx
    type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];
    /*
    [P in keyof T] - T는 객체로 가정하기 때문에 P는 T 객체의 키값을 의미
    Record<P, T[P]> - P타입을 키로 가지고 value는 P를 키로 둔 T 객체의 값 레코드 타입
    { [P in keyof T]: Record<P, T[P]> } 
    - 키는 T 객체의 키 모음. value는 해당 키의 원본 객체  T
    마지막에 다시 [keyof T]의 키값으로 접근하기 때문에 최종 결과는 전달받은 T와 같음
    */
    ```
    
    - `ExcludeOne<T>`
    
    ```tsx
    type ExcludeOne<T> = { 
    	[P in keyof T]: Partial<Record<Exclude<keyof T, P>, undefined>>
    }[keyof T];
    /*
    [P in keyof T] - T는 객체로 가정하기 때문에 P는 T 객체의 키값을 의미
    Exclude<keyof T, P> - T객체가 가진 키값에서 P타입과 일치하는 값을 제외함(A타입이라 가정)
    Record<A, undefined> - 키로 A타입을, 값으로 undefined 타입을 갖는 레코드 타입
    -> 전달 받은 객체 타입을 모두 {[key]:undefined} 형태로 만듦(B타입이라 가정)
    Partial<B> - B타입을 옵셔널로 만듦
    -> {[key]?:undefined} 와 같음 
    최종적으로 [P in keyof T]로 매핑된 타입에서 동일한 객체의 키값인 
    [keyof T]로 접근하기 때문에 Partial<B> 타입이 반환됨
    */
    ```
    
- 결론적으로 얻고자 하는 타입은 속성 하나와 나머지는 옵셔널+undefined인 타입이므로 앞의 속성을 활용하여 PickOne 타입 표현이 가능함
    
    ```tsx
    type PickOne<T> = One<T> & ExcludeOne<T>;
    // One<T> & ExcludeOne<T> 는 [P in keyof T]를 공통으로 가지므로 아래와 같음
    [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>
    // 전달된 T타입의 1개의 키는 값을 가지고 있으며
    // 나머지 키는 옵셔널한 undefined 값을 가진 객체를 의미함
    ```
    

**📌 PickOne 타입 적용하기**

- PickOne 타입을 활용해서 앞의 코드 수정
    
    ```tsx
    type Card = {
    	card: string
    };
    type Account = {
    	account: string
    };
    // PickOne 커스텀 함수구현
    type PickOne<T> = {
      [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
    }[keyof T];
    
    type CardOrAccount = PickOne<Card & Account>; 
    
    function withdraw(type:CardOrAccount){
    	// ...
    }
    withdraw({ card: "hyundai", account: "hana" }); // error
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7d5bae4c-3419-4df1-a2d1-d33c9f921884/f8dd1e59-022d-460d-a5c7-39f42cb881c1/image.png)
    
    - withdraw({ card: "hyundai", account: "hana" }) 부분에서 타입 에러가 발생
- 이처럼 유틸리티 타입으로 원하는 타입을 추출하기 어려울 때 커스텀 유틸리티 타입을 구현
- 커스텀 유틸리티 타입 구현 팁
    - 정확히 어떤 타입을 구현해야하는지 파악하기
    - 필요한 타입을 작은 단위로 쪼개어 생각하여 단계적으로 구현하기

### 3. NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

- null을 가질 수 있는 값의 null 처리는 자주 사용되는 타입 가드 패턴 중 하나
- 일반적으로 if문을 사용해서 null 처리 타입 가드를 적용함
- is 키워드와 NonNullable 타입으로 타입 검사를 위한 유틸 함수를 만들 수도 있음

**📌 NonNullable 타입이란**

- 타입스크립트에서 제공하는 유틸리티 타입
- **제네릭으로 받는 T가 null 또는 undefined일 때 never 또는 T를 반환하는 타입**
- NonNullable을 통해 null이나 undefined가 아닌 경우를 제외할 수 있음
    
    ```tsx
    type NonNullable<T> = T extends null | undefined ? never : T;
    ```
    

**📌 null, undefiend를 검사해주는 NonNullable 함수**

- NonNullable 유틸리티 타입을 통해 타입 가드 함수를 만들 수 있음
- NonNullable 함수
    - 매개변수인 value가 null 또는 undefined라면 false를 반환
    - is키워드로 인해 NonNullable 함수를 사용하는 쪽에서 true가 반환된다면 넘겨준 인자는 null이나 undefined가 아닌 타입으로 타입 가드가 됨(타입이 좁혀짐)
    
    ```tsx
    function NonNullable<T>(value: T): value is NonNullable<T> {
    	return value !== null && value !== undefined;
    }
    ```
    

**📌 Promise.all을 사용할 때 NonNullable 적용하기**

- 예시
    
    ```tsx
    // shopNumber(상품번호)에 따라 응답값을 반환하는 광고 조회 API
    class AdCampaignAPI {
      static async operating(shopNo: number):Promise<AdCampaign[]>{
        try{
          return await fetch(`/ad/shopNumber=${shopNo}`);
        }catch(error){
          return null;
          // 여러 상품의 광고 조회시 하나의 API에서 에러가 발생한다고 해서
          // 전체 광고가 보이지 않으면 안되니 null 반환 
        }
      }
    }
    // AdCampaignAPI를 사용해서 여러 상품의 광고를 받아오는 로직(Promise.All 사용)
    const shopList = [
      {shopNo:100,category:'chicken'},
      {shopNo:101,category:'pizza'},
      {shopNo:102,category:'noodle'},
    ];
    
    const shopAdCampaignList = await Promise.all(shopList.map((shop) 
    	=> AdCampaignAPI.operating(shop.shopNo))
    );
    ```
    
    - `AdCampaignAPI.operating`  함수에서 null을 반환할 수도 있기 때문에 
    shopAdCampaignList의 타입은 **`Array<AdCampaign[] | null>`**
    - shopAdCampaignList 변수를 순회할 때(e.g map or forEach) 고차 함수 내부 콜백함수에서 if을 사용한 타입 가드를 반복하게됨
    - NonNullable을 사용하지 않고 단순하게 필터링할 경우
        
        **[shopAdCampaignList.filter((shop)=>!!shop)]** 타입은 Array<AdCampaign[]>이 아닌 `Array<AdCampaign[] | null>` 로 추론됨.
        
    - NonNullable 사용하기
        
        ```tsx
        const shopAdCampaignList = await Promise.all(shopList.map((shop) 
        	=> AdCampaignAPI.operating(shop.shopNo))
        );
        const shopAds = shopAdCampaignList .filter(NonNullable);
        // shopAds의 타입은 Array<AdCampaign[]>으로 추론됨
        ```
        

## 5.4 불변 객체 타입으로 활용하기

- 상숫값을 관리할 때 객체를 사용
e.g. theme 객체(프로젝트의 전체적인 스타일을 관리), 상숫값을 담은 객체 등
- 컴포넌트나 함수에서 이런 객체를 사용할 때 열린타입으로 설정할 수 있음
- 예시 - 함수인자로 key를 받아서 value를 반환하는 함수
    
    ```tsx
    const colors = {
      red: '#ff0000',
      green:'#186a3b',
      blue:'#2471a3'
    };
    
    const getColorHex = (key: string)=> colors[key];
    //Element implicitly has an 'any' type because expression of type 'string' 
    // can't be used to index type '{ red: string; green: string; blue: string; }'.
    ```
    
    - 키 타입을 해당 객체에 존재하는 키 값이 아닌 string으로 설정하면 colors에 어떤 값이 추가될지 모르기 때문에  getColorHex의 반환 값의 타입은 any가 됨
    - 이런 경우 `as const` 키워드로 객체를 불변 객체로 선언하고 `keyof` 연산자로 getColorHex 함수 인자로 실제 colors 객체에 존재하는 키 값만 받도록 설정 가능

### 1. Atom 컴포넌트에서 theme style 객체 활용하기

- Atom 컴포넌트(e.g. Button, Header, Input 등)는 폰트 크기, 색상 등 다양한 환경에서 유연하게 사용될 수 있도록 구현해야하며 관련 설정값은 props로 넘겨주도록 설계함
    
    하지만 props를 통해 값을 직접 넘겨줄 경우 사용자가 모든 색상 값을 인지해야 하며 변경 사항이 생길 때마다 모두 수정해야 함
    
    → **이런 문제를 해결하기 위해 해당 프로젝트의 스타일 값을 관리하는 theme 객체로 관리**
    
- **Atom 컴포넌트에서는 theme 객체의 색상, 폰트 사이즈의 키값을 props로 받은 뒤 theme 객체에서 값을 받아오도록 설계함**
- 예시 - 컴포넌트에서 props의 color, fontSize 값의 타입을 string 으로 설정한 경우
    
    ```tsx
    interface Props{
      fontSize ?: string;
      backgroundColor?: string;
      color?: string;
      onClick:(event:React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
    }
    
    const Button: FC<Props> = ({fontSize, backgroundColor, color, children }) => {
    	return (
    		<ButtonWrap
    			fontSize={fontSize}
    			backgroundColor={backgroundColor}
    			color={color}
    		>
    			{children }
    		</ButtonWrap>
    	);
    };
    
    const ButtonWrap = styled.button<Omit<Props, "onClick">>`
    	color: ${({color})=> theme.color[color ?? "default"]};
    	background-color: ${({backgroundColor})=> 
    		theme.backgroundColor[backgroundColor?? "default"]};
    	font-size: ${({fontSize})=> theme.fontSize[fontSize?? "default"]};
    `;
    ```
    
    - Button 컴포넌트의 props로 color, backgroundColor 등을 넘겨줄 때 키 값이 자동 완성되지 않음
    - 잘못된 키값을 넣어도 에러가 발생하지 않음
        
        **⇒ theme 객체로 타입을 구체화해서 해결 가능**
        

### 타입스크립트 keyof 연산자로 객체의 키값을 타입으로 추출하기

- `keyof` 연산자
    - 객체 타입을 받아 해당 객체의 키값을 string 또는 number의 리터럴 유니온 타입을 반환
    - 객체 타입으로 인덱스 시그니처가 사용되었다면, 인덱스 시그니처의 키 타입 반환
- ColorType 객체 타입의 keyof ColorType을 사용하면 객체의 키 값들이 유니온으로 나오게 됨
    
    ```tsx
    interface ColorType {
        red:string;
        green:string;
        blue:string;
    }
    
    type ColorKeyType = keyof ColorType; // 'red' | 'green' | 'blue'
    ```
    

### 타입스크립트 typeof 연산자로 값을 타입으로 다루기

- keyof 연산자는 객체 타입을 받음 
→ 객체의 키값을 타입으로 다루기 위해선 값 객체를 타입으로 변환 필요. 이 때 typeof 연산자를 활용
- 자바스크립트의 typeof
    - 타입을 추출하기 위한 연산자로 사용
- 타입스크립트의 typeof
    - **변수 혹은 속성의 타입을 추론하기 위해 사용**
    - 주로 ReturnType같이 유틸리티 타입이나 keyof 연산자같이 타입을 받는 연산자와 함께 쓰임
- typeof로 colors 객체의 타입을 추론한 결과
    
    ```tsx
    const colors = {
      red: '#ff0000',
      green:'#186a3b',
      blue:'#2471a3'
    };
    type ColorsType = typeof colors;
    /*
    {
    	red: string;
      green: string;
      blue: string;
    }
    */
    ```
    

### 객체의 타입을 활용해서 컴포넌트 구현하기

- keyof, typeof 연산자를 통해 theme 객체 타입을 구체화하고 string 타입을 설정했던 Button 컴포넌트 개선하기
    
    ```tsx
    import React, { FC } from "react";
    import styled from "styled-components";
    
    const colors = {
      black: "#000000",
      gray: "#222222",
      white: "#FFFFFF",
      mint: "#2AC1BC",
    };
    
    const theme = {
      colors: {
        default: colors.gray,
        ...colors
      },
      backgroundColors: {
        default: colors.white,
        gray: colors.gray,
        mint: colors.mint,
        black: colors.black,
      },
      fontSize: {
        default: "16px",
        small: "14px",
        large: "18px",
      },
    };
    
    type ColorType = keyof typeof theme.colors;
    type BackgroundColorType = keyof typeof theme.backgroundColors;
    type FontSizeType = keyof typeof theme.fontSize;
    
    interface Props {
      color?: ColorType;
      backgroundColor?: BackgroundColorType;
      fontSize?: FontSizeType;
      children?: React.ReactNode;
      onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
    }
    
    const Button: FC<Props> = ({ fontSize, backgroundColor, color, children }) => {
      return (
        <ButtonWrap
          fontSize={fontSize}
          backgroundColor={backgroundColor}
          color={color}
        >
          {children}
        </ButtonWrap>
      );
    };
    
    const ButtonWrap = styled.button<Omit<Props, "onClick">>`
      color: ${({ color }) => theme.colors[color ?? "default"]};
      background-color: ${({ backgroundColor }) =>
        theme.backgroundColors[backgroundColor ?? "default"]};
      font-size: ${({ fontSize }) => theme.fontSize[fontSize ?? "default"]};
    `;
    ```
    
    - 이처럼 여러 상숫값을 인자나 props로 받은 후 객체의 키 값을 추출한 타입을 활용하면 객체에 접근시  타입스크립트의 도움을 받아 실수를 방지할 수 있음

## 5.5 Record 원시 타입 키 개선하기

- **`Record<Keys, Type>`**
    - Record는 Keys로 이루어진 object타입을 만드는데, 각 프로퍼티들이 전달받은 Type이 됨
    
    ```tsx
    interface CatInfo {
      age: number;
      breed: string;
    }
     
    type CatName = "miffy" | "boris" | "mordred";
     
    const cats: Record<CatName, CatInfo> = {
      miffy: { age: 10, breed: "Persian" },
      boris: { age: 5, breed: "Maine Coon" },
      mordred: { age: 16, breed: "British Shorthair" }
    };
    // cats는 CatName으로 이루어진 Object
    // 각 프로퍼티들은 CatInfo 타입
    ```
    
- 객체 선언 시 키가 명확하지 않다면 Record의 키를 string이나 number 같은 원시타입으로 명시하곤 함
→ 이 경우 타입스크립트는 키가 유효하지 않아도 타입상 문제가 없기 때문에 오류 표시를 하지 않지만 런타임 에러가 발생할 수 있음
- Record를 명시적으로 사용하는 방안에 대해 알아보자

### 1. 무한한 키를 집합으로 가지는 Record

- 예시
    
    ```tsx
    type Category = string; // 원시 타입으로 선언 
    interface Food {
      name: string;
    }
    const foodByCategory: Record<Category, Food[]> = {
      한식:[{name:'제육덮밥'}, {name:'뚝불'}],
      일식:[{name:'초밥'}, {name:'라멘'}],
    }
    
    console.log(foodByCategory['한식'])
    console.log(foodByCategory['양식']) // Food[]로 추론
    foodByCategory['양식'].map((food)=> console.log(food.name));
    // 위코드는 컴파일시 타입에러 발생 X
    // 하지만 런타임에서 undefined가 되어 오류를 반환
    ```
    
    - string 타입인 Category를 Record의 키로 사용하는 foodByCategory 객체는 무한한 키 집합을 가짐
    - foodByCategory 객체에 없는 키값을 사용하더라도(e.g. 양식) 타입스크립트는 오류를 표시하지 않음. 하지만 런타임에서 undefined가 되어 오류를 반환하게됨
    - 이 경우 자바스크립트의 옵셔널 체이닝 등을 사용하면 런타임 에러를 방지할 수 있음
    → 그러나 undefined일 수 있는 값을 일일이 판단해서 적용해야하므로 불편함

### 2. 유닛 타입으로 변경하기

- 키가 유한한 집합이라면 유닛타입을 사용하면 됨
    
    ```tsx
    type Category = '한식" | "일식";
    // ...
    console.log(foodByCategory['양식']);
    // error: Property '양식' does not exist on type 'Record<Category, Food[]>'
    ```
    
- 키가 무한해야 하는 상황에는 부적합

### 3. Partial을 활용하여 정확한 타입 표현하기

- **`Partial<Type>`**
    - 전달 받은 Type의 모든 하위 집합을 나타내는 타입을 생성
    - 특정 타입의 부분 집합을 만족하는 타입 정의 가능
    
    ```tsx
    interface Address {
      email: string;
      address: string;
    }
    
    type MyEmail = Partial<Address>;
    const me: MyEmail = {}; // ok
    const you: MyEmail = { email: "you@gmail.com" }; // ok
    const all: MyEmail = { email: "all@gmail.com", address: "seoul" }; // ok
    ```
    
- 키가 무한한 경우 Partial을 사용하면 해당 값이 undefind일 수 있는 상태임을 표현 가능
- 객체 값이 undefined일 수 있는 경우 Partial을 사용해서 PartialRecord 타입을 선언하고 객체 선언시 활용
     ```tsx
    type PartialRecord<K extends string, T> = Partial<Record<K, T>>;
    type Category = string;
    interface Food {
        name: string;
    }
    
    const foodByCategory: PartialRecord<Category, Food[]> = {
        한식:[{name:'제육덮밥'}, {name:'뚝불'}],
        일식:[{name:'초밥'}, {name:'라멘'}],
    }
    
    console.log(foodByCategory['한식']);
    console.log(foodByCategory['양식']); // Food[] or undefined 타입으로 추론
    foodByCategory['양식'].map((food)=> console.log(food.name));
    // object is possibly 'undefined'
    foodByCategory['양식']?.map((food)=> console.log(food.name));
    // 타입스크립트가 개발자에게 값을 처리하라고 표시해줌
    // 옵셔널 체이닝 or 조건문을 사용하면 사전에 조치 가능
    ```
