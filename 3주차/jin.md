## 4장. 타입 확장하기/좁히기

- **타입 확장** : 기존 타입을 이용해서 새로운 타입을 정의
    - 장점
        - 불필요한 코드 중복 제거
        - 명시적인 코드 작성
        - 확장성 : 필요한 요구사항이 생길 때마다 필요한 타입 손쉽게 생성
    - 유니온 타입 (합집합)
        - 2개 이상의 타입을 조합
        
        ```tsx
        type MyUnion = A | B
        ```
        
        - 유니온 타입으로 선언된 값은 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근 가능
            - 타입 안전성을 보장하기 위해
            - `step`이 `CookingStep`인지 `DeliveryStep`인지 정확히 알 수 없는 상황에서, TypeScript가 안전하게 접근할 수 있는 속성만 허용하기 위해
            - 타입 좁히기를 통해 에러 해결 가능
        
        ```tsx
        interface CookingStep {
          orderId: string;
          price: number;
        }
        
        interface DeliveryStep {
          orderId: string;
          time: number;
          distance: string;
        }
        
        function getDeliveryDistance(step: CookingStep | DeliveryStep) {
          return step.distance;
          // Property ‘distance’ does not exist on type ‘CookingStep | DeliveryStep’
          // Property ‘distance’ does not exist on type ‘CookingStep’
        }
        ```
        
    - 교차 타입 (교집합)
        - 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입 생성
        - 모든 속성을 가진 단일 타입, 두 타입의 공통 부분을 찾음
        
        ```tsx
        type MyIntersection = A & B;
        ```
        
    - extends와 교차타입
        - 유사한 결과를 얻을 수 있지만 코드의 구조와 상속/결합 방식에서 차이가 존재
    - 유니온 타입과 교차타입을 사용한 새로운 타입은 오직 `type` 키워드로만 선언이 가능

- **타입 좁히기** : 타입 범위를 더 좁은 범위로 좁혀나가는 과정
    - 장점
        - 명시적인 타입 추론 가능
        - 타입 안정성 증가
    - 타입 가드 : 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능
        - 자바스크립트 연산자 사용 : 런타임에 유효한 타입 가드
            - `typeof`
                - 원시타입 추론 시
                - `string, number, boolean, undefined, object, function, bigint, symbol`
            - `instanceof`
                - 인스턴스화된 객체 타입 판별 시
                - `타입을 검사할 대상 변수 instanceof 특정 객체의 생성자`
            - `in`
                - 객체의 속성 유무 판별 시
                - `속성 in 객체` : 객체 내 속성 존재하는지
        - 사용자 정의 타입 가드 : 사용자가 직접 어떤 타입으로 값을 좁힐지 직접 지정
            - `is`
                - 타입 명제 : 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수
                    - `매개변수 is 타입`
                    - 일반 함수의 반환값을 통한 분기는 타입 안전성을 보장하지 못하기에 `is`를 사용하는 것이 효과적
    - 식별할 수 있는 유니온 (태그된 유니온)
        - 에러 정의하기
            - 타입 구분 명확 ⇒ 정확한 타입 검증 가능
            - 불필요한 속성 추가 방지
        
        ```tsx
        type TextError = {
          errorCode: string;
          errorMessage: string;
        };
        type ToastError = {
          errorCode: string;
          errorMessage: string;
          toastShowDuration: number; // 토스트를 띄워줄 시간
        };
        type AlertError = {
          errorCode: string;
          errorMessage: string;
          onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
        };
        
        type ErrorFeedbackType = TextError | ToastError | AlertError;
        const errorArr: ErrorFeedbackType[] = [
          { errorCode: “100”, errorMessage: “텍스트 에러” },
          { errorCode: “200”, errorMessage: “토스트 에러”, toastShowDuration: 3000 },
          { errorCode: “300”, errorMessage: “얼럿 에러”, onConfirm: () => {} },
        ];
        
        const errorArr: ErrorFeedbackType[] = [
          // ...
          {
          errorCode: “999”,
          errorMessage: “잘못된 에러”,
          toastShowDuration: 3000,
          onConfirm: () => {},
          }, // expected error
        ];
        
        // 타입 간의 구분이 모호
        // errorCode와 errorMessage 속성만으로는 ToastError와 AlertError를 구별하기 어려움
        // 올바르지 않은 속성(toastShowDuration이 TextError에 포함되거나 onConfirm이 ToastError에 포함되는 등)을 실수로 넣더라도 TypeScript가 이를 감지하지 못할 수 있습니다.
        ```
        
        ```tsx
        //식별할 수 있는 유니온
        // 공통적인 식별자 필드 (errorType)을 추가하여 타입 구분 가능
        type TextError = {
          errorType: “TEXT”;
          errorCode: string;
          errorMessage: string;
        };
        type ToastError = {
          errorType: “TOAST”;
          errorCode: string;
          errorMessage: string;
          toastShowDuration: number;
        }
        type AlertError = {
          errorType: “ALERT”;
          errorCode: string;
          errorMessage: string;
          onConfirm: () = > void;
        };
        
        type ErrorFeedbackType = TextError | ToastError | AlertError;
        
        const errorArr: ErrorFeedbackType[] = [
          { errorType: “TEXT”, errorCode: “100”, errorMessage: “텍스트 에러” },
          {
            errorType: “TOAST”,
            errorCode: “200”,
            errorMessage: “토스트 에러”,
            toastShowDuration: 3000,
          },
          {
            errorType: “ALERT”,
            errorCode: “300”,
            errorMessage: “얼럿 에러”,
            onConfirm: () => {},
          },
          {
            errorType: “TEXT”,
            errorCode: “999”,
            errorMessage: “잘못된 에러”,
            toastShowDuration: 3000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘TextError’
            onConfirm: () => {},
          },
          {
            errorType: “TOAST”,
            errorCode: “210”,
            errorMessage: “토스트 에러”,
            onConfirm: () => {}, // Object literal may only specify known properties, and ‘onConfirm’ does not exist in type ‘ToastError’
          },
          {
            errorType: “ALERT”,
            errorCode: “310”,
            errorMessage: “얼럿 에러”,
            toastShowDuration: 5000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘AlertError’
          },
        ];
        ```
        
        - 공통 속성 vs 식별 가능한 유니온
            - 단순히 공통 속성만 가지는 것으로 충분하지 않고 **각 타입을 구별할 수 있는 식별자(고유한 값)** 가 필요
                - 유닛 타입(오직 하나의 정확한 값을 가지는 타입)으로 선언
                    - `null, undefined, 리터럴 타입, true, 1` 등
    - Exhaustiveness Checking 으로 정확한 타입 분기 유지
        - Exhaustiveness Checking : 모든 케이스에 대해 철저하게 타입을 검사
        
        ```tsx
        type ProductPrice = “10000” | “20000” | “5000”;
        
        const getProductName = (productPrice: ProductPrice): string => {
          if (productPrice === “10000”) return “배민상품권 1만 원”;
          if (productPrice === “20000”) return “배민상품권 2만 원”;
          // if (productPrice === “5000”) return “배민상품권 5천 원”;
          else {
            exhaustiveCheck(productPrice); // Error: Argument of type ‘string’ is not assignable to parameter of type ‘never’
            return “배민상품권”;
          }
        };
        
        const exhaustiveCheck = (param: never) => {
          throw new Error(“type error!”);
        };
        ```
        

> [!IMPORTANT]
never 타입 : 절대 발생하지 않는 값, 어떤 값도 할당할 수 없는 타입 ⇒ 타입 안전성 유지, 잘못된 코드 흐름 방지
사용 사례 : 절대 반환되지 않는 함수, 유니온 타입에서 모든 경우를 처리한 후 남는 불가능한 상태, 타입 좁히기 후 발생할 수 없는 타입
>
