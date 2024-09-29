# 3장 - 고급 타입

## 3.1 타입스크립트만의 독자적 타입 시스템

- 이번 장에서 나오는 모든 타입 시스템은 타입스크립트에만 존재하는 키워드
    
    하지만 개념은 자바스크립트에 기인한 타입 시스템
    
- 타입스크립트의 타입 계층 구조

    ![image](https://github.com/user-attachments/assets/b9a9302a-9fc1-42a1-8c2c-7aaec6d61de1)


### 1. any 타입

- 자바스크립트에 존재하는 모든 값을 오류 없이 받을 수 있음
    
    →자바스크립트에서의 기본적인 사용 방식과 같으므로 타입을 명시하지 않은 것과 동일한 효과
    
- any 타입을 사용하는 것은 지양해야함
- 타입스크립트에서 any 타입을 어쩔 수 없이 사용해야하는 경우
    - 개발 단계에서 임시로 값을 지정해야 할 때
    - 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
        - 외부 라이브러리 등을 사용할 경우 어떤 인자를 주고 받을지 특정하기 힘들 수 있음
    - 값을 예측할 수 없을 때 암묵적으로 사용
        - 외부 라이브러리나 웹 API 요청에 따라 다양한 값을 반환하는 API가 있을 수 있음
        e.g. 브라우저의 Fetch API
    
    📌 예외적으로 any 타입을 사용해야 하는 상황이 있지만 any 타입은 최대한 지양할 것!
    
- any 타입을 사용하면 잘못된 타입의 값을 사용할 경우 타입스크립트 컴파일러에서는 에러가 도출되지 않지만, 실제 런타임에서 심각한 오류가 발생할 수 있음

### 2. unknown 타입(타입스크립트 3.0에 추가)

- 기존 타입 시스텝에서 부족한 부분을 보완하기 위해 등작
- any 타입과 유사하게 모든 타입의 값 할당 가능
- any 외 타입으로 선언된 변수에는 unknown 타입 값 할당 불가능
- any 타입과 unknown 타입 비교
    
    
    | any | unknown |
    | --- | --- |
    | 어떤 값이든 any 타입에 할당 가능 | 어떤 타입이든 unknwon 타입에 할당 가능 |
    | any 타입은 어떤 타입으로도 할당 가능(never 제외)| unknown 타입은 any 타입 외 타입으로 할당 불가능 |
     
    
    ```tsx
    let unknownValue: unknown;
    unknownValue = 100; // 숫자 할당 가능 
    unknownValue = "hello"; // 문자열 할당 가능 
    unknownValue = () => console.log("hello"); // 함수 할당 가능
    // 위 케이스의 경우 any 타입과 유사함
    
    let SomeValue1: any = unknownValue; // 정상 동작
    // any 타입으로 선언된 변수를 제외한 다른 변수는 할당 불가
    let SomeValue2: number = unknownValue;
    // 에러 Type 'unknown' is not assignable to type 'number'.
    let SomeValue3: string = unknownValue;
    // Type 'unknown' is not assignable to type 'string'.
    ```
    
- any 타입과 비슷해보이는 unknown 타입이 추가된 이유

    ![image](https://github.com/user-attachments/assets/7cc03099-fae9-45cc-b409-2803c5ef815e)

    - 함수를 unknown 타입 변수에 할당할 때는 에러가 발생하지 않지만 호출시 에러가 발생함
    (함수뿐만 아니라 객체의 속성 접근, 클래스 생성자 호출을 통한 인스턴스 생성 등 객체 내부에 접근하는 모든 시도에서 에러가 발생)
    - unknown 타입은 어떤 타입이 할당되었는지 앖 수 없음을 나타냄. 
    unknown 타입으로 선언된 변수는 값을 가져오거나 내부 속성 접근이 불가능  <br />
     → **unknown 타입으로 할당된 변수는 어떤 값이든 올 수 있지만 개발자에게 엄격한 타입 검사를 강제하는 의도를 담고 있음**
    - any타입을 사용하면 어떤 값이든 허용되는데 이로 인해 런타임에 예상치 못한 버그가 발생할 가능성이 높아짐 <br />
      → 이런 상황을 보완하기 위해 unknown 타입이 등장
- 타입 검사를 강제하고 타입이 식별된 후에 사용할 수 있기 때문에 any 타입보다 더 안전  <br />
→ 데이터 구조를 파악하기 힘들 때 any 타입 대신 unknown 타입으로 대체해서 사용하는 것이 좋음

### 3. void 타입

- void 타입 사용 예시
    
    ```tsx
    // 반환 값이 없는 함수들 
    function showModal(type: ModalType): void {
    	feedbackSlice.actions.createModal(type);
    }
    
    // 화살표 함수로 작성
    const ShowModal = (type: ModalType):void => {
    	feedbackSlice.actions.createModal(type);
    };
    ```
    
- 자바스크립트는 함수에서 명시적인 반환문을 작성하지 않으면 기본적으로 undefined를 반환
- 타입스크립트에서의 void 타입은 undefined가 아님
- 타입스크립트는 함수가 어떤 값을 반환하지 않는 경우 void를 지정해서 사용
- void 타입은 주로 함수 반환 타입으로 사용됨 <br />
    → 함수 뿐만 아니라 변수에도 할당이 가능하지만 잘 사용되지 않음
    
- void 타입으로 지정된 변수에는 undefined 또는 null 값만 할당할 수 있음 <br />
→ 하지만 undefined와 null에는 각 타입 키워드를 직접 사용해서 지정하는 것이 바람직함

### 4. never 타입

- 함수와 관련하여 많이 사용되는 타입
- **never 타입은 값을 반환할 수 없는 타입을 뜻함**
    
    → 값을 반환하지 않는 것과 반환하는 것을 명확하게 구분할 것
    
- 자바스크립트에서 값을 반환할 수 없는 경우
    - 에러를 던지는 경우
        
        ```tsx
        function generateError(res:Response): naver {
        	throw new Error(res.getMessage());
        }
        ```
        
        - 자바스크립트에서 throw 키워드를 사용하면 에러를 발생시킬 수 있는데,
        이 경우 값을 반환하는 것으로 간주하지 않음
            
            → **특정 함수가 실행 중 마지막에 에러를 던지는 작업을 수행한다면 해당 함수의 반환 타입은 never**
            
    - 무한히 함수가 실행되는 경우
        
        ```tsx
        function checkStatus(): never {
        	while(true) {
        		// ...
        	}
        }
        ```
        
        - 함수 내에서 무한 루프를 실행하는 경우
        - 무한 루프는 함수가 종료되지 않음을 의미 → 값을 반환하지 못함
- never 타입은 모든 타입의 하위 타입
    
    → never 자신을 제외한 어떤 타입도 never에 할당될 수 없음(any 타입도 할당 불가능)
    
- 타입스크립트에서는 조건부 타입을 결정할 때 특정 조건을 만족하지 않는 경우 엄격한 타입 검사를 목적으로 never 타입을 명시적으로 사용하기도 함
- never 타입 사용 예시
    - 허용할 수 없는 함수 매개변수에 제한을 가하는 경우
    - `switch`, `if-else` 문의 모든 상황을 보장하는 경우
        - switch 문에서 default case로 사용하면 남아있는 건 `never` 타입이이어야 하기 때문에 모든 상황에 대처하는 것을 보장할 수 있음
        
        ```tsx
        function unknownColor(x: never): never {
          throw new Error("unknown color");
        }
        
        type Color = 'red' | 'green' | 'blue'
        
        function getColorName(c: Color): string {
          switch(c) {
            case 'red':
              return 'is red';
            case 'green':
              return 'is green';
            default:
              return unknownColor(c); 
              // 'string' 타입은 'never' 타입에 할당할 수 없음
          }
        }
        ```
        
    - 타이핑을 부분적으로 허용하지 않음
        - 타입스크립트는 구조적 서브 타이핑을 기반으로 하기 때문에 객체 리터럴을 전달하지 않는 한 파라미터의 타입보다 더 많은 속성을 가진 객체를 함수에 전달하는 것은 허용됨
            
            ```tsx
            type VariantA = {
              a: string,
            }
            
            type VariantB = {
              b: number,
            }
            
            declare function fn(arg: VariantA | VariantB): void
            
            const input = {a: 'foo', b: 123 }
            fn(input) // 타입 오류가 발생하지 않음
            ```
            
        - `never` 를 통해 구조적 타이핑을 비활성화하고 사용자가 두 속성을 모두 포함하는 객체를 전달하지 못하도록 할 수 있음
            
            ```tsx
            type VariantA = {
                a: string
                b?: never
            }
            
            type VariantB = {
                b: number
                a?: never
            }
            
            declare function fn(arg: VariantA | VariantB): void
            
            const input = {a: 'foo', b: 123 }
            fn(input) // 에러 발생
            ```
            
            ![image](https://github.com/user-attachments/assets/78b8efd0-2059-43b8-9d10-932a0f3491dd)

    
    [타입스크립트의 Never 타입 완벽 가이드](https://ui.toast.com/posts/ko_20220323)
    

### 5. Array 타입

- 자바스크립트에서의 배열 타입
    - `Object.prototype.toString.call(...)` 함수
        
        ```tsx
        const arr = [];
        console.log(typeof(arr)); // 'object'
        console.log(Object.prototype.toString.call(arr)); // '[object Array]'
        ```
        
        - 함수의 객체 타입을 알아내는 데 사용
        → 객체의 인스턴스까지 알 수 있음
        - 참고로 `typeof` 의 경우 객체 타입을 단순히 object 타입으로만 알려줌
- 자바스크립트에서도 확인 가능한 자료형임에도 불구하고 타입스크립트에서 배열을 다루는 이유
    - 자바스크립트에서는 배열을 객체에 속하는 타입으로 분류함
        
        → 배열을 단독으로 배열이라는 자료형에 국한하지 않음
        
    - 타입스크립트에서 Array라는 타입을 사용하기 위해서는 타입스크립트의 특수한 문법을 함께 다뤄야함
- 타입스크립트에서는 일반적으로 **배열의 크기까지 제한하진 않지만** 정적 타입의 특성을 살려 **명시적인 타입을 선언하여 해당 타입의 원소를 관리하는 것을 강제함**
- 배열 타입을 선언하는 방식
    - 자료형 + 대괄호([]) 형식
        
        ```tsx
        const array: number[] = [1,2,3]
        ```
        
    - 제네릭 사용
        
        ```tsx
        const arr: Array<number> = [1,2,3]
        ```
        
    - 두 방식은 선언하는 형식 외에는 차이점이 없음
- 여러 타입을 모두 관리해야 하는 배열 선언시 `유니온 타입`을 사용
    
    ```tsx
    const array1: Array<number|string> = [1, 'string'];
    const array2: number[] | string[] = [1, 'string'];
    // 후자의 방식은 아래와 같이 선언할 수도 있음
    const array3: (number|string)[] = [1, 'string'];
    ```
    
- 튜플
    - 배열 타입의 하위 타입으로 타입스크립트의 배열 기능에 길이 제한까지 추가한 타입시스템
    - 선언 방식
        
        ```tsx
        let tuple: [number] = [1];
        tuple = [1, 2]; // 불가능
        tuple = [1, 'string']; // 불가능
        
        let tuple: [number, string, boolean] = [1, 'string', true]; 
        // 여러 타입과 혼합 가능
        ```
        
        - 타입 시스템과 대괄호를 사용해서 선언
            
            → **대괄호 안에 타입 시스템을 기술**
            
        - 대괄호 안에 선언하는 타입의 개수가 튜플이 가질 수 있는 원소의 개수를 뜻함
            
            → 튜플은 배열의 특정 인덱스에 정해진 타입을 선언하는 것
            
- 배열은 사전에 허용하지 않은 타입이 서로 섞이는 것을 방지하여 타입 안정성을 제공함
- 튜플은 길이까지 제한하여 원소 개수와 타입을 보장하여 더 안정적인 타입 안정성을 제공
- 튜플 예시
    - `리액트의 useState`
        
        ```tsx
        import {useState} from 'react';
        const [value, setValue] = useState(false);
        const [username, setUsername] = useState('');
        ```
        
        - `useState`는 튜플 타입을 반환함
        첫번째 원소 : 훅으로 부터 생성 및 관리되는 상태 값
        두번째 원소 : 해당 상태를 조작할 수 있는 setter
        - 배열 원소의 자리마다 명확한 의미를 부여하기 때문에 컴포넌트에서 사용하지 않은 값에 접근하는 오류 방지 가능
        - 구조 분해 할당을 통해 사용자가 자유롭게 이름을 정의 할 수 있음

### 6. enum 타입

- 타입스크립트에서 지원하는 특수한 타입
- 일종의 구조체를 만드는 타입시스템
- enum을 통해 열거형 정의가 가능함
    - **타입스크립트의 enum은 명명한 각 멤버의 값을 스스로 추론함**
    - 기본적인 추론 방식 → 숫자 0부터 1씩 늘려가며 값을 할당
    
    ```tsx
    enum ProgramingLanguage {
    	Typescript, // 0
    	Javascript, // 1
    	Java, // 2
    	Python, // 3
    }
    // 접근하는 방식은 자바스크립트에서 객체의 속성에 접근하는 방식과 동일
    ProgramingLanguage.Typescript; // 0
    ProgramingLanguage['Java']; // 2
    // 역방향으로도 접근이 가능함
    ProgramingLanguage[3]; // 'Python'
    ```
    
    - 각 멤버에 **명시적으로 값의 할당이 가능**
    → 일부 멤버에 값을 직접 할당하지 않을 경우 타입스크립트가 이전 멤버 값의 숫자를 기준으로 1씩 늘려가며 자동 할당함
    
    ```tsx
    enum ProgramingLanguage {
    	Typescript = 'Typescript', 
    	Javascript = 'Javascript',
    	Java = 300,
    	Python, // 301
    }
    ```
    
- enum 타입은 주로 문자열 상수를 생성하는 데 사용
- enum 타입은 그 자체로 변수 타입으로 지정이 가능함
    
    ```tsx
    enum ItemStatusType {
    	DELIVERY_HOLD = 'DELIVERY_HOLD', // 배송 보류
    	DELIVERY_READY = 'DELIVERY_READY', // 배송 준비중
    	DELIVERED = 'DELIVERED', // 배송 완료
    }
    const checkItemAvailable = (itemStatus:ItemStatusType) => {
    	switch(itemStatus){
    		case ItemStatusType.DELIVERY_HOLD:
    		case ItemStatusType.DELIVERY_READY:
    			return false;
    		case ItemStatusType.DELIVERED :
    		default:
    			return true;
    	}
    };
    ```
    
    - checkItemAvailable 함수의 인자인 itemStatus는 ItemStatusType 열거형을 타입으로 가짐
    - itemStatus의 타입이 문자열로 지정된 경우와 비교했을 때,
    타입 안정성이 우수하며 높은 응집력과 가독성을 가짐
- enum 타입 사용 주의점
    - 숫자로만 이루어져있거나 타입스크립트가 자동으로 추론한 열거형의 경우 안전하지 않은 결과를 낳을 수 있음
        - 할당된 값을 넘어서 역방향으로 접근하는 경우 타입스크립트는 에러를 발생시키지 않음
        - 위와 같은 동작을 막기위해  `const enum` 으로 열거형을 선언할 수 있음
        
        ```tsx
        ProgramingLanguage[200]; // undefined 출력. 에러는 발생하지 않음
        const enum ProgramingLanguage {
        	// ...
        }
        ```
        
        - 숫자 상수로 관리되는 열거형의 경우 `const enum` 으로 선언하더라도 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못함
            
            **→  숫자 상수 방식보다 문자열 상수 방식으로 열거형을 사용하는 것이 안전함**
            
    - enum 타입은 타입 공간과 값 공간에서 모두 사용됨
        - enum 타입을 사용한 타입스크립트 코드가 자바스크립트로 변환될 때 즉시 실행 함수(IIFE) 형식으로 변환됨
        이때 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있음
        → 즉, 불필요한 코드의 크기가 증가할 가능성 존재
            
            ```tsx
            enum ProgramingLanguage {
            	Typescript = "Typescript", 
            	Javascript = "Javascript",
            	Java = 300,
            	Python, // 301
            }
            const favoritLanguage:ProgramingLanguage = ProgramingLanguage.Javascript;
            ```
            
            ![image](https://github.com/user-attachments/assets/566452ec-e47d-4054-994f-76b904acf9d6)

            ⇒ 위와 같은 문제는 `const enum` 또는 `as const assertion` 을 사용해서 유니온 타입으로 열거형과 동일한 효과를 얻는 방식을 통해 해결할 수 있음
            

## 3.2 타입 조합

### 1. 교차 타입(Intersection)

- 교차 타입을 통해 여러 가지 타입을 결합하여 하나의 단일 타입으로 만들 수 있음
    
    → 기존 타입들을 합쳐서 해당 타입의 모든 멤버를 가지는 새로운 타입을 생성 
    
- `&` 을 통해 표기
- 결과물로 탄생한 단일 타입에는 타입 별칭을 붙일 수 있음
    
    ```tsx
    type Person = {
      name:string,
      age:number,
    };
    type PersonInfo = Person & { hobby: string };
    ```
    

### 2. 유니온 타입(Union)

- 타입 A 또는 타입 B 중 하나가 될 수 있는 타입을 뜻함
- `A | B` 와 같이 표기
- 특정 변수가 가질 수 있는 타입을 전부 나열하는 용도로 사용
    
    ```tsx
    type Customer = {
    	id:string,
    	name:string,
    	age:number,
    }
    type Someone = Person | Customer;
    // Someone은 Person 혹은 Customer가 될 수 있는 유니온 타입
    ```
    

### 3. 인덱스 시그니처(Index Signatures)

- 특정 타입의 속성 이름은 알 수 없지만 속성 값의 타입을 알고 있을 때 사용하는 문법
- 인터페이스 내부에 `[Key: K]: T` 와 같이 타입을 명시
    
    → 해당 타입의 속성 키는 모두 K타입이어야 하고 속성값은 모두 T 타입을 가져야한다는 의미
    
    ```tsx
    interface IndexSignatureEx {
    	[key:string]: number;
    }
    ```
    

### 4. 인덱스드 엑세스 타입(Indexed Access Types)

- 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용
- 인덱스에 사용되는 타입 또한 그 자체로 타입
→ 유니온 타입, keyof, 타입 별칭 등의 표현 사용 가능
    
    ```tsx
    type Example = {
    	a: number;
    	b: string;
    	c: boolean;
    };
    
    type IndexedAccess = Example['a']; // number
    type IndexedAccess2 = Example['a' | 'b']; // number | string
    type IndexedAccess3 = Example[keyof Example]; // number | string | boolean
    type ExAlias = 'b' | 'c';
    type IndexedAccess4 = Example[ExAlias]; // string | boolean
    ```
    

### 5. 맵드 타입(Mapped Types)

- 다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법
- 인덱스 시그니처 문법을 사용해서 반복적인 타입 선언을 효과적으로 줄일 수 있음
    
    ```tsx
    type Example = {
    	a: number;
    	b: string;
    	c: boolean;
    };
    
    type Subset<T> = {
    	[K in keyof T]?: T[K];
    };
    
    const aExample: Subset<Example> = {a:3};
    const bExample: Subset<Example> = {b:'hello'};
    const acExample: Subset<Example> = {a:4, c:true};
    ```
    
- 만약 인덱스 시그니처 문법을 사용하지 않는다면?
    - 모든 속성을 수동으로 나열해야함
    - Example 타입에 속성이 추가되거나 변경될 경우 SubsetWithoutIndexSignature도 수정필요
    
    ```tsx
    type Example = {
    	a: number;
    	b: string;
    	c: boolean;
    };
    
    type SubsetWithoutIndexSignature = {
    	a?: number;
    	b?: string;
    	c?: boolean;
    };
    
    const aExample: SubsetWithoutIndexSignature = { a: 3 };
    const bExample: SubsetWithoutIndexSignature = { b: 'hello' };
    const acExample: SubsetWithoutIndexSignature = { a: 4, c: true };
    ```
    
- 맵드 타입에서 매핑할 때는 `readonly`와 `?`를 수식어로 적용할 수 있음
    - `readonly` : 읽기 전용으로 만들고 싶은 때 붙여주는 수식어
    - `?` : 선택적 매개변수(옵셔널 파라미터)로 만들고 싶을 때 붙여주는 수식어
- 맵드 타입은 위의 수식어를 더해주는 것뿐만 아니라 제거도 가능
    - 기존 타입에 존재하던 `readonly`나 `?` 앞에 `-` 를 붙여주면 해당 수식어를 제거한 타입 선언이 가능함
- 맵드 타입에서는 `as 키워드` 를 통해 키의 재지정이 가능

### 7. 템플릿 리터럴 타입(Template Literal Types)

- 자바스크립트의 템플릿 리터럴 문자열을 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법
    
    ```tsx
    type Stage = 
    	| 'init'
    	| 'select-image'
    	| 'edit-image'
    	| 'decorate-card'
    	| 'capture-image'
    type StateName = `${Stage}-Stage`;
    // "init-Stage" | "select-image-Stage" | "edit-image-Stage" | 
    // "decorate-card-Stage" | "capture-image-Stage"
    ```
    
    - Stage 타입의 모든 유니온 멤버 뒤에 -stage를 붙여서 새로운 유니온 타입을 생성

### 8. 제네릭(Generic)

- 정적 언어에서 다양한 타입 간에 재사용성을 높이기 위해 사용하는 문법
- `일반화된 데이터 타입`
함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워둔 후 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정하여 사용하는 방식
    
    → 여러 타입에 대해 하나하나 따로 정의하지 않아도 되기 때문에 재사용성이 크게 향상됨
    
- 타입 변수는 일반적으로 `<T>` 와 같이 꺽쇠괄호 내부에 정의됨
- 사용시 함수에 매개변수를 넣는 것과 유사하게 원하는 타입을 넣어줌
    - 보통 타입 변수명으로 `T(Type)` , `E(Elemnt)` , `K(Key)` , `V(Value)` 등 한 글자로 된 이름을 많이 사용함
    
    ```tsx
    type ExampleArrayType<T> = T[];
    const array1 = ExampleArrayType<string> = ['치킨', '피자', '라면'];
    ```
    
- 제네릭은 `any` 처럼 아무 타입이나 무분별하게 받는 것이 아니라 **배열 생성 시점에 원하는 타입으로 특정할 수 있음**
    
    → 제네릭 사용시 배열 요소가 전부 동일한 타입이라고 보장 가능
    
- 제네릭 함수 호출시 꺽쇠괄호(<>) 안에 타입을 명시하지 않아도 컴파일러가 인수를 통해 타입 추론 가능
- 특정 요소 타입을 알 수 없을 때 제네릭 타입에 기본값 추가 가능
- 제네릭 사용시 주의점
    - 파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러가 발생함
        - tsx는 타입스크립트 + JSX이므로 제네릭의 꺽쇠괄호와 태그의 꺽쇠괄호를 혼동하여 문제가 생김
        - 제네릭 부분에 `extends 키워드`를 사용하여 해결 가능
    
    ```tsx
    // 에러 발생 : JSX element 'T' has no corresponding closing tag
    const arrowExampleFunc = <T>(arg: T): T[] => {
    	return new Array(3).fill(arg);
    };
    
    // 에러 발생 X
    const arrowExampleFunc2 = <T extends {}>(arg: T): T[] => {
    	return new Array(3).fill(arg);
    };
    ```
    

## 3.3 제네릭 사용법

### 1. 함수의 제네릭

- 어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭 사용 가능
    
    ```tsx
    function ReadOnlyRepository<T>(target: ObjectType<T> | EntityShema<T> | string);
    Repository<T> {
    	return getConnection('ro').getRepository(target);
    }
    // T 자리에 넣는 타입에 따라 ReadOnlyRepository가 적절하게 사용될 수 있음 
    ```
    

### 2. 호출 시그니처의 제네릭

- 호출 시그니처
    - 타입스크립트의 함수 타입 문법
    - 함수의 매개변수와 반환 타입을 미리 선언하는 것
    - 개발자가 함수 호출시 필요한 타입을 별도로 지정할 수 있음
- 호출 시그니처를 사용할 때 제네릭 타입을 어디에 위치시키는지에 따라 타입의 범위와 제네릭 타입을 언제 구체 타입으로 한정할지 결정 가능
    - 우아한형제들 배민선물하기 팀의 호출 시그니처 제네릭 활용 예시
    
    ```tsx
    interface useSelectPaginationProps<T> {
    	categoryAtom: RecilState<nubmer>;
    	filterAtom: RecoilState<string[]>; 
    	sortAtom:	RecoilState<SortType>;
    	fetcherFunc: (props: CommonListRequest) => Promise<DefaultResponse<ContentListREsponse<T>>>;
    }
    ```
    
    **📎 코드 설명**
  
    `<T>`는 useSelectPaginationProps의 타입 별칭으로 한정
    따라서 useSelectPaginationProps을 사용할 때 타입을 명시함으로써 제네릭 타입을 구체 타입으로 한정함
    useSelectPaginationProps가 사용되는 useSelectPagination 훅의 반환 값도 인자에서 쓰는 제네릭 타입은 T와 연관있기 때문에 이렇게 작성됨.
    

### 3. 제네릭 클래스

- 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스
    
    ```tsx
    class LocalDB<T> {
    	// ...
    	async put(table: string, row: T): Promise<T> {
    		return new Promise<T>((resolved, rejected)=> {
    			// T 타입의 데이터를 DB에 저장
    		})
    	}
    	async get(table: string, key: any): Promise<T> {
    		return new Promise<T>((resolved, rejected)=> {
    			// T 타입의 데이터를 DB에서 가져옴
    		})
    	}
    	async getTable(table: string): Promise<T[]> {
    		return new Promise<T[]>((resolved, rejected)=> {
    			// T[] 타입의 데이터를 DB에서 가져옴
    		})
    	}
    }
    
    export default class IndexedDB implements ICacheStore {
    	private _DB?: LocalDB<{ key:string; value: Promise<Record<string, unknown>>;
    	cacheTTL: number }>;
    	
    	private DB(){
    		if(!this._DB){
    			this._DB = new LocalDB('localCache', {
    			ver:6, 
    			tables:[{name: TABLE_NAME, keyPath: 'key'}] });
    		}
    		return this._DB;
    	}
    	// ...
    }
    ```
    
    **📎 코드 설명**
  
    클래스 이름 뒤에 타입 매개변수인 `<T>` 를 선언
    `<T>` 는 메서드의 매개변수나 반환 타입으로 사용될 수있음 
    LocalDB 클래스는 외부에서
    
    `{ key:string; value: Promise<Record<string, unknown>>; cacheTTL: number }`
    
    타입을 받아들여 클래스 내부에서 사용될 제네릭 타입으로 결정됨
    
- 제네릭 클래스를 사용하면 클래스 전체에 걸쳐 타입 매개변수가 적용됨
특정 메서드만을 대상으로 제네릭을 적용하려면 해당 메서드를 제네릭 메서드로 선언하면됨.

### 4. 제한된 제네릭

- 타입 매개변수에 대한 제약 조건을 설정하는 기능
- 타입 매개변수 T 타입을 제약하는 방법
    - e.g. string 타입으로 제약하려면 매개변수 특정 타입을 상속(extends)해야함
    
    ```tsx
    type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never 
    ? Partial<Record<Key, boolean>>
    : never; 
    ```
    
    - 이처럼 타입 매개변수가 특정 타입으로 묶였을 때(bind) 키를 바운드 타입 매개변수라고 부르며, string을 키의 상한 한계라고 함.

### 5. 확장된 제네릭

- 제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수 있음
    - `<Key extends string>`
      
      위처럼 타입을 제약해버리면 제네릭의 유연성을 잃어버림.
    - `<Key extends string | number>`
        
        제네릭의 유연성을 잃지 않고 타입을 제약할 때는 타입 매개변수에 유니온 타입을 상속해서 선언하면됨.
        

### 6. 제네릭 예시

- 실제 현업에서 제네릭을 가장 많이 활용하는 경우는 **API 응답 값의 타입을 지정할 때**
    - 실제 코드 예시
    
    ```tsx
    export interface MobileApiResponse<Data> {
    	data: Data;
    	statusCode: string;
    	statusMessage?: string;
    }
    
    export const fetchPriceInfo = (): Promise<MobileApiResponse<PriceInfo>> => {
    	const priceUrl = 'https: ~~~'; // url 주소
    	return request({
    		method: 'GET',
    		url: priceUrl ,
    	});
    };
    export const fetchOrderInfo = (): Prpmise<MobileApiResponse<Order>> => {
    	const orderUrl = 'https: ~~~'; // url 주소
    	return request({
    		method: 'GET',
    		url: priceUrl ,
    	});
    };
    ```
    
    **📎 코드 설명**
  
    API 응답값에 따라 달라지는 data를 제네릭 타입 Data로 선언하고 있음. 
    이렇게 만든 MobileApiResponse는 실제 API 응답 값의 타입을 지정할 때 사용되며,
    다양한 API응답값의 타입에 MobileApiResponse을 활용해서 코드의 재사용이 가능해짐
    
- 제네릭을 굳이 사용하지 않아도 되는 타입
    
    ```tsx
    type GType<T> = T;
    type RequirementType = 'USE' | 'UN_USE' | 'NON_SELECT';
    interface Order {
    	getRequirement(): GType<RequirementType>;
    }
    
    // 위 코드는 아래처럼 사용하는 것과 동일함
    type RequirementType = 'USE' | 'UN_USE' | 'NON_SELECT';
    interface Order {
    	getRequirement():RequirementType;
    }
    ```
    
    - GType이 다른 곳에서 사용되지 않고 getRequirement 함수의 반환 값 타입으로만 사용된다고 가정했을 때 
    GType이라는 이름이 목적의 의미를 정확히 담고 있지도 않고, 
    제네릭을 사용하지 않고도 타입 매개변수를 그대로 선언하는 것과 같은 기능을 하고 있으므로 제네릭을 사용하지 않는 것이 나음.
- any 사용하기
    - any를 사용하면 제네릭을 포함해 타입을 지정하는 의미가 사라짐
- 가독성을 고려하지 않은 사용
    - 제네릭이 과하게 사용되면 가독성을 해침
    - 복잡한 제네릭은 의미 단위로 분할해서 사용할 것
