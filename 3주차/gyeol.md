# 4장 - 타입 확장하기/좁히기

## 4.1 타입 확장하기

- interface와 type 키워드를 사용해서 타입을 정의
- extends, 교차 타입, 유니온 타입을 사용해서 타입을 확장

### 1. 타입 확장의 장점

- 타입 확장을 통해 코드의 중복을 줄일 수 있음
- 명시적인 코드 작성이 가능함
- 확장성이 좋으므로 코드 작성시 효율적임
- 타입 확장 예시
    - **`interface` 키워드 사용**
        
        ```tsx
        /**
         * 메뉴 요소 타입
         * 메뉴 이름, 이미지, 할인율, 재고 정보를 담고 있다.
         **/
        interface BaseMenuItem {
        	itemName: string | null;
        	itemImageUrl: string | null;
        	itemDiscountAmount: number;
        	stock: number | null;
        }
        /**
         * 장바구니 요소 타입
         * 메뉴 타입에 수량 정보가 추가되었다.
         **/
        interface BaseCartItem extends BaseMenuItem {
        	quantity: number;
        }
        ```
        
    - **`type` 키워드 사용**
        
        ```tsx
        type BaseMenuItem = {
        	itemName: string | null;
        	itemImageUrl: string | null;
        	itemDiscountAmount: number;
        	stock: number | null;
        };
        type BaseCartItem = {
        	quantity: number;
        } & BaseMenuItem ;
        ```
        

### 2. 유니온 타입

- 2개 이상의 타입을 조합하여 사용하는 방법
- 유니온 타입은 집합 관점에서 합집합과 비슷
    
    ```tsx
    type MyUnion = A | B;
    ```
    
    - A와 B의 유니온 타입인 MyUnion은 타입 A와 타입 B의 합집합
    - A타입과 B타입의 모든 값이 MyUnion 타입의 값이 됨
- **유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근 가능**

### 3. 교차 타입

- 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것으로 이해 가능
- 교차 타입은 집합 관점에서 교집합과 비슷
    
    ```tsx
    type MyInterSection = A & B;
    ```
    
    - MyInterSection 타입의 모든 값은 A 타입의 값이며 MyInterSection 타입의 모든 값은 B 타입의 값
    - 집합 MyInterSection 의 모든 원소는 집합 A의 원소이자 집합 B의 원소

```tsx
// 교차 타입 예시
interface CookingStep {
	orderId: string;
	time: number;
	price: number;
}
interface DeliveryStep {
	orderId: string;
	time: number;
	distance: string;
}

type BaedalProgress = CookingStep & DeliveryStep;
function logBeadalInfo(progress: BaedalProgress ){
	console.log(`주문 금액: ${progress.price}`);
	console.log(`배달 거리: ${progress.distance}`);
}
```

- B**aedalProgress는 CookingStep과 DeliveryStep타입을합쳐 모든 속성을 가진 단일 타입**
    
    **→ BaedalProgress타입의 progress값은 CookingStep이 갖고있는 price 속성과 DeliveryStep이 갖고 있는 distance의 속성을 포함하고 있음.**
    

>  📌 **참고**
> 
> 타입스크립트의 타입을 속성의 집합이 아닌 **값의 집합으로 이해**해야함! <br />
> BaedalProgress  교차 타입은 CookingStep이 가진 속성과 DeliveryStep이 가진 속성을 모두 만족(교집합)하는 값의 타입(집합)이라고 해석 가능

- 교차 타입 사용시 타입이 서로 호환되는 경우 예시
    
    ```tsx
    /* 배달 팁 */
    interface DeliveryTip {
    	tip: string;
    }
    /* 별점 */
    interface StarRating {
    	rate: number;
    }
    /* 주문 필터 */
    type Filter = DeliveryTip & StarRating ;
    
    const filter: Filter = {
    	tip: '1000원 이하',
    	rate: 4,
    };
    ```
    
    - DeliveryTip과 StarRating은 공통된 속성이 없는데도 FIlter의 타입은 공집합(never 타입)이 아닌 DeliveryTip과 StarRating의 속성을 모두 포함한 타입이 됨
    → **타입이 속성이 아닌 값의 집합으로 해석되기 때문**
- 교차 타입 사용시 타입이 서로 호환되지 않는 경우 예시
    
    ```tsx
    type IdType = string | number;
    type Numeric = number | boolean;
    
    type Universal = IdType & Numeric ;
    ```
    
    - Universal 타입이 가질 수 있는 경우의 수
        - string이면서 number인 경우
        - string이면서 boolean인 경우
        - **number이면서 number인 경우**
        - number이면서 boolean인 경우
    - Universal은 IdTyperhk Numeric의 교차타입 이므로 두 타입을 모두 만족하는 경우에만 유지됨
    **→ number이면서 number인 경우만 유효하기 때문에 Universal의 타입은 number가 됨**

### 4. extends와 교차 타입

- extends 키워드를 사용해서 교차 타입을 작성할 수 있음
    
    ```tsx
    interface BaseMenuItem {
    	itemName: string | null;
    	itemImageUrl: string | null;
    	itemDiscountAmount: number;
    	stock: number | null;
    }
    
    interface BaseCartItem extends BaseMenuItem {
    	quantity: number;
    }
    ```
    
    - BaseCartItem은 BaseMenuItem을 확장함으로써 BaseMenuItem의 속성을 모두 포함함
    → BaseCartItem은 BaseMenuItem의 속성을 모두 포함하는 상위 집합. BaseMenuItem은 BaseCartItem의 부분 집합이 됨.
- 위 코드를 교차 타입의 관점에서 작성하면 아래와 같음
    
    ```tsx
    type BaseMenuItem = {
    	itemName: string | null;
    	itemImageUrl: string | null;
    	itemDiscountAmount: number;
    	stock: number | null;
    };
    type BaseCartItem = {
    	quantity: number;
    } & BaseMenuItem ;
    
    const baseCartItem: BaseCartItem = {
    	itemName: '겨리네 떡볶이',
    	itemImageUrl: 'hppts://www.~~',
    	itemDiscountAmount: 2000,
    	stock: 100,
    	quantity: 2,
    };
    ```
    
    - **유니온 타입과 교차 타입을 사용한 새로운 타입은 `type` 키워드로만 선언할 수 있음**
- **`extends` 키워드를 사용한 타입은 `교차 타입`과 100% 상응하지 않음**
    
    ```tsx
    // extedns 키워드를 사용한 타입
    interface DeliveryTip {
      tip: number;
    }
    
    interface Filter extends DeliveryTip {
      tip: string; // error 발생!
      // Interface 'Filter' incorrectly extends interface 'DeliveryTip'.
      // Types of property 'tip' are incompatible.
      // Type 'string' is not assignable to type 'number'.
    }
    ```
    
    - DeliveryTip 타입은 number 타입의 tip 속성을 가지고 있음
    - 이 때 DeliveryTip을 extends 키워드를 통해 확장한 Filter 타입에 string 타입 속성 tip을 선언하면 tip의 타입이 호환되지 않는 다는 에러가 발생함
    
    ```tsx
    // 교차 타입
    type DeliveryTip = {
    	tip: number;
    };
    
    type Filter = DeliveryTip  & {
    	tip: string; // 에러가 발생하지 않음
    };
    ```
    
    - **선언 시 에러가 발생하지 않지만 해당 타입을 사용해서 변수를 선언하면 에러가 발생함
    (number, string 둘 다 에러가 발생)**

        ![image](https://github.com/user-attachments/assets/5b115ef9-82d6-4b7e-bd9e-5556101f6d64)

        ![image](https://github.com/user-attachments/assets/c66a0c7e-5168-4622-9510-43126f1d9384)

        
    - **Filter 타입의 tip 속성의 타입은 `never`**
    - **`type` 키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문에 에러가 발생하지 않음**
    - **하지만 tip이라는 같은 속성에 대해 서로 호환되지 않는 타입이 선언되어 `never` 타입이 됨**
    

---

## 📌 참고. 타입스크립트에서 권장하는 컴파일 되기 쉬운 코드 작성법
- Preferring Interfaces Over Intersections(교차 타입보다 인터페이스를 선호하기)

- 한 객체 유형에 대한 단순한 타입의 경우 인터페이스와 비슷하게 작동함
    
    ```tsx
    // 아래의 두 코드는 비슷하게 작동
    interface Foo { prop: string }
    type Bar = { prop: string };
    ```
    
- 두 개 이상의 타입을 구성해야 할 경우 아래의 방법을 통해 구성할 수 있음
    - `interface` 와 `extends` 키워드를 통해 구성하기
    - `type` 으로 교차타입 구성하기
        
        → 두 방법은 확실한 **차이**가 존재하며, 성능 또한 차이가 있음.
        
    - 타입 확장 예시
        
        ```tsx
        // 교차 타입 구성하기
        type Foo = Bar & Baz & {
          someProp: string;
        }
        // interface, extends로 구성하기
        interface Foo extends Bar, Baz {
          someProp: string;
        }
        ```
        

### interface와 type의 차이

1. **타입의 속성 머지**
    
    > *“Interfaces create a single flat object type that detects property conflicts, which are usually important to resolve! Intersections on the other hand just recursively merge properties, and in some cases produce never.”*
    > 
    - `interface`
        - 속성 간 충돌을 해결하기 위해 단순한 객체 타입을 생성
            
            → interface는 객체의 타입을 만들기 위한 것. 어차피 객체만 오기 때문에 단순히 합치기만 하면 되기 때문
            
    - `type`
        - 재귀적으로 순회하며 속성을 머지함
        → type은 interface와 달리 원시 타입이 올 수 있음. 충돌이 나서 제대로 머지가 안 되는 경우 `never` 가 떨어짐
        
        ```tsx
        type type2 = { a: 1 } & { b: 2 }; // 잘 머지됨
        type type3 = { a: 1; b: 2 } & { b: 3 }; // resolved to `never`
        
        const t2: type2 = { a: 1, b: 2 }; // good
        const t3: type3 = { a: 1, b: 3 }; // Type 'number' is not assignable to type 'never'.(2322)
        const t3: type3 = { a: 1, b: 2 }; // Type 'number' is not assignable to type 'never'.(2322)
        ```
        
    - **타입 간 속성을 머지할 땐 주의가 필요하며, 객체에서만 쓰는 용도라면 `interface` 를 사용하는 것이 나음**
2. **타입 관계 캐싱**
    
    > *“Type relationships between interfaces are also cached, as opposed to intersection types as a whole.“*
    > 
    - interface 간의 타입 관계는 캐시되어 재사용이 가능
    
    > *“A final noteworthy difference is that when checking against a target intersection type, every constituent is checked before checking against the "effective"/"flattened" type.”*
    > 
    - 교차(intersection) 타입 합성 자체에 대한 유효성을 판단하기 전에 모든 구성요소에 대한 타입을 체크하므로 컴파일 시 상대적으로 성능이 좋지 않음

[Performance](https://github.com/microsoft/Typescript/wiki/Performance#preferring-interfaces-over-intersections)

[타입스크립트 type과 interface의 공통점과 차이점](https://yceffort.kr/2021/03/typescript-interface-vs-type)

---

### 5. 배달의민족 메뉴 시스템에 타입 확장 적용하기

```tsx
interface Menu {
  name: string;
  image: string;
}
```

- 요구 사항
    - 특정 메뉴를 길게 누르면 gif 파일이 재생되어야한다.
    - 특정 메뉴는 이미지 대신 별도의 텍스트만 노출되어야 한다.
- **[방법1] 하나의 타입에 여러 속성을 추가하기
:** 기존 Menu 인터페이스에 추가된 정보를 전부 추가
    
    ```tsx
    interface Menu {
      name: string;
      image: string;
      gif?: string;
      text?: string;
    }
    
    const menuList: Menu[] = [
      {name:"찜", image:"찜.png"},
      {name:"찌개", image:"찌개.png"},
      {name:"회", image:"회.png"},
    ];
    
    const specialMenuList: Menu[] = [
      {name:"돈까스", image:"돈까스.png", gif:"돈까스.gif"},
      {name:"피자", image:"피자.png",gif:"피자.gif"},
    ];
    const packageMenuList: Menu[] = [
      {name:"1인분", image:"1인분.png", text:"1인 가구 맞춤형"},
      {name:"족발", image:"족발.png", text:"맛있는 족발"},
    ];
    
    const textList = specialMenuList.map((menu)=> menu.text); 
    // 책에서는 TypeError: Cannot read properties of undefined 
    // 라는 에러가 발생한다고 하지만 테스트해봤을 땐 발생하지 않았음
    // 그냥 undefined가 반환된다.
    console.log(textList) // [undefined, undefined] 
    ```
    
    - specialMenuList는 Menu 타입의 원소를 갖기 때문에 text 속성에 접근 가능
    - specialMenuList 배열의 모든 원소는 text라는 속성을 가지고 있지 않으므로 에러가 발생
    → 실제로 테스트 해봤을 때 text 속성이 없으면 에러는 발생하지 않고 undefined가 반환됨. 
    챗지피티한테 물어보니 속성이 존재하지 않으면 undefined가 반환되고 에러는 발생하지 않고, 객체 자체가 undefined일 때 해당 객체의 속성에 접근하려고 하면 TypeError가 발생한다고 함
- **[방법2] 타입을 확장하는 방식**
: 기존 Menu 인터페이스는 유지한 채 각 요구 사항에 따른 별도 타입을 만들어서 확장
    
    ```tsx
    interface Menu {
      name: string;
      image: string;
    }
    // gif를 활용한 메뉴 타입
    interface SpecialMenu extends Menu {
    	gif: string;
    }
    // 별도의 텍스트를 활용한 메뉴 타입
    interface PackageMenu extends Menu {
    	text: string;
    }
    
    const specialMenuList: SpecialMenu[] = [
      {name:"돈까스", image:"돈까스.png", gif:"돈까스.gif"},
    ];
    const packageMenuList: PackageMenu[] = [
      {name:"1인분", image:"1인분.png", text:"1인 가구 맞춤형"},
    ];
    // 아래의 경우 에러가 발생함 
    const specialMenuList2: Menu[] = [
      {name:"돈까스", image:"돈까스.png", gif:"돈까스.gif"},
    ];
    // Object literal may only specify known properties, 
    // and 'gif' does not exist in type 'Menu'.
    
    const textList = specialMenuList.map((menu)=> menu.text);
    // Property 'text' does not exist on type 'SpecialMenu'
    ```
    
    - 위 방법은 각 배열의 타입을 확장할 타입에 맞게 명확히 규정할 수 있음
    - specialMenuList 배열의 원소 내 속성에 동일하게 접근한다고 가정하면 프로그램을 실행하지 않고도 타입이 잘못되었음을 미리 알 수 있음
- 주어진 타입에 무분별하게 속성을 추가하여 사용하는 것 보다 **타입을 확장해서 사용하는 것이 나음**
    
    → 적절한 네이밍을 사용해서 타입의 의도를 명확히 표현할 수 있고, 코드 작성 단계에서 버그 예방이 가능하기 때문
    

## 4.2 타입 좁히기 - 타입 가드

- 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정
- 이를 통해 더 정확하고 명시적인 타입 추론이 가능해짐 → 타입 안정성을 높일 수 있음

### 1. 타입 가드에 따라 분기 처리하기

- 분기 처리
    - 조건문과 타입 가드를 통해 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따라 다른 동작을 수행하는 것
- 타입 가드
    - 런타임에 조건문을 사용하여 타입을 검사하고 타임 범위를 좁혀주는 기능
- 어떤 함수가 A | B 타입의 매개변수를 받는다고 가정했을 때 인자 타입이 A 또는 B일 때를 구분해서 로직을 처리하고 싶다면?
    - ~~if문을 사용해서 처리~~
        
        → 컴파일 시 타입 정보는 모두 제거되어 런타임에 존재하지 않기 때문에 타입을 사용하여 조건을 만드는 건 불가능함
        
    - 특정 문맥 안에서 타입스크립트가 해당 변수를 타입 A로 추론하도록 유도하면서 런타임에서도 유효한 방법이 필요함 ⇒ 이럴 때 `타입 가드`를 사용하면 됨
- 타입 가드
    - 자바스크립트 연산자를 활용한 타입 가드
        - **`typeof` , `instanceof` , `in`** 과 같은 연산자를 사용해서 제어문으로 특정 타입 값을 가질 수 밖에 없는 상황을 유도하여 타입을 좁히는 방식
        - **런타임에 유효한 타입 가드를 만들기 위해 자바스크립트 연산자를 이용**
    - 사용자 정의 타입 가드
        - 사용자가 어떤 타입으로 값을 좁힐지 직접 지정하는 방식

### 2. 원시 타입을 추론할 때 : `typeof` 연산자 활용하기

- `typeof`는 자바스크립트 타입 시스템만 대응이 가능함
- 자바스크립트의 동작 방식으로 인해 null과 배열 타입 등이 object 타입으로 판별되는 등 복잡한 타입을 검증하는 데엔 한계가 있음
    
    **→ `typeof` 연산자는 주로 원시 타입을 좁히는 용도로만 사용할 것**
    
- `typeof` 연산자를 사용해서 검사할 수 있는 타입 목록
    - string
    - number
    - boolean
    - undefined
    - object
    - function
    - bigint
    - symbol

### 3. 인스턴스화된 객체 타입을 판별할 때 : `instanceof` 연산자 활용하기

- **A instanceof B** 형태로 사용
- A : 타입을 검사할 대상 변수 / B : 특정 객체의 생성자
- **`instanceof`는 A의 프로토타입 체인에 생성자 B가 존재하는지 검사 후 존재한다면 true, 그렇지 않다면 false를 반환함**
- **A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있다는 점**을 유의해야함
    
    ```tsx
    // selected 매개변수가 Date인지 검사 후 Range 타입의 객체를 반환할 수 있도록 분기처리
    interface Range {
    	start: Date;
    	end: Date;
    }
    interface DatePickerProps {
    	selectedDates?: Date | Range;
    }
    const DatePicker = ({selectedDates}: DatePickerProps) => {
    	const [selected, setSelected] = useState(convertToRange(selectedDates));
    	// ...
    };
    export function convertToRange(selected?: Date | Range;): Range | undefined {
    	return selected instanceof Date
    		? { start: selected, end: selected }
    		: selected;
    }
    ```
    

### 4. 객체의 속성이 있는지 없는지에 따른 구분 : `in` 연산자 활용하기

- 객체에 속성이 있는지 없는지 확인 후 true 또는 false를 반환
- **A in B**의 형태로 사용 → A라는 속성이 B 객체에 존재하는지를 검사
- 프로토타입 체인으로 접근할 수 있는 속성이면 전부 true를 반환
- B 객체 내부에 A 속성이 있는지 없는지를 검사하기 때문에 **B 객체에 존재하는 A 속성에 undefined를 할당한다고 해서 false를 반환하는 것은 아님**.
→ delete 연산자를 통해 객체 내부에서 해당 속성을 제거해야만 false를 반환함
- **자바스크립트의 in 연산자는 런타임의 값만 검사하지만, 타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사함**
- 여러 객체 타입을 유니온 타입으로 가지고 있을 때 in 연산자를 통해 속성의 유무에 따라 조건 분기 가능

### 5. `is` 연산자로 사용자 정의 타입 가드 만들어 활용하기

- 반환 타입이 타입 명제인 함수를 정의해서 사용 가능
- **A is B** 형식으로 타입 명제를 작성
- A : 매개변수 이름 / B : 타입
- **참/거짓의 진릿값을 반환하면서 반환 타입을 타입 명제로 지정하면 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급하게 됨**
- 타입 명제(type predicates)
: 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수
- is 연산자 사용 예시
    
    ```tsx
    interface Cat {
      meow: number;
      fly: boolean;
    }
    interface Bird {
      chirp: number;
      fly: boolean;
    }
    
    // 타입 가드 역할을 하는 커스텀 함수
    // Cat 타입인지 확인 하는 역할을 한다. (리턴 타입에 is 키워드를 붙여 사용한다.)
    function isCat(animal: Cat | Bird): animal is Cat {
      // 직접 타입 판별 로직 구현 (meow라는 프로퍼티가 있다면)
      if ((animal as Cat).meow) {
        // meow, fly 자동완성
        return true; // 고양이라면 true
      } else {
        return false; // 새라면 false
      }
    }
    
    function pet(animal: Cat | Bird) {
      if (isCat(animal)) {
        // 고양이일 경우
        console.log(animal.meow); // meow, fly 자동완성
      } else {
        // 새일 경우
        console.log(animal.chirp); // chirp, fly 자동완성
      }
    }
    ```
    
    - isCat 함수의 리턴 타입을 boolean 으로 지정하면?
        
        ```tsx
        function isCat(animal: Cat | Bird): boolean {
        	// ...
        }
        function pet(animal: Cat | Bird) {
          if (isCat(animal)) {
            console.log(animal.meow);
            // Property 'meow' does not exist on type 'Cat | Bird'.
            // Property 'meow' does not exist on type 'Bird'.
          } else {
            console.log(animal.chirp); 
            // Property 'chirp' does not exist on type 'Cat | Bird'.
            // Property 'chirp' does not exist on type 'Cat'.
          }
        }
        ```
        
        - animal의 타입을 좁히지 못하므로 pet 함수의 매개변수인 animal의 타입은 'Cat | Bird' 가 되어버림
        - animal에는 Cat과 Bird의 교집합인 fly 프로퍼티만 들어갈 수 있게되므로 animal.meow와 animal.chirp를 사용할 경우 에러가 발생함

## 4.3 타입 좁히기 - 식별할 수 있는 유니온(Descriminated Unions)

### 1. 에러 정의하기

- 에러 타입을 아래와 같이 정의하고 에러타입의 유니온 타입을 원소로 하는 배열을 정의
    
    ```tsx
    type TextError = {
    	errorCode: string;
    	errorMessage: string;
    };
    type ToastError = {
    	errorCode: string;
    	errorMessage: string;
    	toastShowDuration: number; //  토스트를 틔워줄 시간
    };
    type AlertError = {
    	errorCode: string;
    	errorMessage: string;
    	onConfirm: () => void; // alert 참의 확인 버튼을 누른 후 액션
    };
    
    type ErrorFeedbackType = TextError | ToastError | AlertError;
    const errorArr : ErrorFeedbackType [] = [
    	{errorCode: '100', errorMessage:'텍스트 에러'},
    	{errorCode: '200', errorMessage:'토스트 에러', toastShowDuration:3000},
    	{errorCode: '300', errorMessage:'얼럿 에러', onConfirm:()=>{}},
    ];
    ```
    
- 아래와 같은 코드를 작성했을 때 타입 에러가 발생하는 것을 기대하지만 에러가 발생하지 않음
    
    ```tsx
    const errorArr: ErrorFeedbackType[] = [
    	{
    		errorCode: '999',
    		errorMessage: '잘못된 에러',
    		toastShowDuration: 3000,
    		 onConfirm:()=>{},
    	}, // expectred error
    ]
    ```
    
    → 자바스크립트는 덕타이핑 언어이기 때문에 별도의 타입 에러를 뱉지않음. 실제로 이런 상황에서 타입에러가 발생하지 않으면 개발 과정에서 의미를 알 수 없는 무수한 에러 객체가 생길 위험이 커짐
    

### 2. 식별할 수 있는 유니온

- 위와 같은 이유로 에러 타입을 구분할 방법이 필요함
- 각 타입이 비슷한 구조를 가지지만 서로 호환되지 않도록 만들어 주기 위해서는 타입들이 서로 포함관계를 가지지 않도록 정의해야함
    
    → 식별할 수 있는 유니온을 활용
    
- 식별할 수 있는 유니온
    - 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함관계를 제거하는 것
- 판별자의 개념으로 errorType이라는 필드를 새로 정의
→ 판별자를 달아주면 이들은 포함관계를 벗어나게됨
    
    ```tsx
    type TextError = {
    	errorType: 'TEXT';
    	errorCode: string;
    	errorMessage: string;
    };
    type ToastError = {
    	errorType: 'TOAST';
    	errorCode: string;
    	errorMessage: string;
    	toastShowDuration: number; //  토스트를 틔워줄 시간
    };
    type AlertError = {
    	errorType: 'ALERT';
    	errorCode: string;
    	errorMessage: string;
    	onConfirm: () => void; // alert 참의 확인 버튼을 누른 후 액션
    };
    type ErrorFeedbackType = TextError | ToastError | AlertError;
    const errorArr: ErrorFeedbackType[] = [
    	{
    		errorType: 'TEXT',
    		errorCode: '999',
    		errorMessage: '잘못된 에러',
    		toastShowDuration: 3000, 
    		// 에러 발생 : Object literal may only specify known properties, 
    		// and 'toastShowDuration' does not exist in type 'TextError'.
    		onConfirm:()=>{},
    	},
    ]
    ```
    
    - 정확하지 않은 에러 객체에 대해 타입 에러가 발생하게됨

### 3. 식별할 수 있는 유니온의 판별자 선정

- 식별할 수 있는 유니온을 사용할 때 주의할 점
    - 식별할 수 있는 유니온의 판별자는 유닛 타입으로 선언되어야 정상적으로 동작함
- 유닛 타입(unit type)
    - 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입
    - unll, undefined, 리터럴 타입을 비롯한 true, 1 등 정확한 값을 나타내는 타입이 유닉타입에 해당
    - 다양한 타입을 할당할 수 있는 void, string, number와 같은 타입은 유닛타입으로 적용 불가능
- 타입스크립트 공식 깃허브 이슈 탭에서는 식별할 수 있는 유니온의 판별자로 사용할 수 있는 타입을 아래와 같이 정의하고 있음
    - 리터럴 타입이어야한다.
    - 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화할 수 있는 타입은 포함되지 않아야한다.

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- `Exhaustiveness Checking`은 모든 케이스에 대해 철저하게 타입을 검사하는 것을 뜻하며 타입 좁히기에 사용되는 패러다임 중 하나
- 문제 상황
    - ProductPrice타입에 ‘5000’ 이 추가되었는데 getProductName 함수를 수정하지 않았을 경우 
    별도의 에러가 발생하지 않아 **개발자가 실수할 여지가 있음.**
    
    ```tsx
    type ProductPrice = '10000' | '20000' | '5000';
    const getProductName = (productPrice:ProductPrice): string => {
    	if(productPrice === '10000') return '상품권 1만 원';
    	if(productPrice === '20000') return '상품권 2만 원';
    	else {
    		return '배민 상품권'
    	}
    }
    ```
    
- 위와 같은 실수를 방지하기 위해 모든 타입에 대한 타입 검사를 강제하고 싶다면?
    
    ```tsx
    type ProductPrice = '10000' | '20000' | '5000';
    const getProductName = (productPrice:ProductPrice): string => {
    	if(productPrice === '10000') return '상품권 1만 원';
    	if(productPrice === '20000') return '상품권 2만 원';
    	// if(productPrice === '5000') return '상품권 5천 원';
    	else {
    		ExhaustiveCheck(productPrice);
    		// 에러 발생 Argument of type 'string' is not assignable to parameter of 
    		// type 'never'.
    		return '배민 상품권'
    	}
    }
    
    // 아래 함수를 타입 처리 조건문의 마지막 else 문에 사용하면 
    // 앞의 조건문에서 모든 타입에 대한 분기처리를 강제할 수 있음
    const ExhaustiveCheck = (param:never) => {
    	// 매개변수를 never 타입으로 선언해서 
    	// 어떤 값도 받을 수 없고 만일 값이 들어온다면 에러를 뱉도록 처리
    	throw new Error('type error!');
    };
    ```
    
    - ProductPrice의 타입 중 ‘5000’이라는 값에 분기처리를 하지 않아서(철저한 검사를 하지 않아서) 에러가 발생함
- 이처럼 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때 컴파일타임 에러가 발생하게 하는 것을 `Exhaustiveness Checking` 이라함
- `Exhaustiveness Checking`을 활용하면 예상치 못한 런타임 에러를 방지하거나 요구사항이 변경되었을 때 생길 수 있는 위험성을 줄일 수 있음
