## 3장 고급 타입
### 3.1 타입스크립트만의 독자적 타입 시스템
1. any 타입
- any 타입은 자바스크립트에 존재하는 모든 값을 오류 없이 받을 수 있음
- any 타입은 타입스크립트의 타입 안전성을 무력화시키는 요소라 지양해야 함

**그럼에도! any 타입 사용 예시**

1. 개발 단계에서 임시로 값을 지정해야 할 때
- 추후 값이 변경될 가능성이 있는 경우 임시 사용

2. 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
- 타입이 잘 정제되지 않은 외부 라이브러리를 사용하여 어떤 인자를 주고받을지 특정하기 힘든 경우

3. 값을 예측할 수 없을 때 암묵적으로 사용
- 외부 라이브러리나 웹 API의 요청에 따라 다양한 값을 반환하는 API가 존재할 수 있음
- 예를 들어 브라우저의 Fetch API
```javascript
async function load(){
    const response = await fetch('https://api.com');
    const data = await response.json(); // response.json()의 반환값은 any 타입
    return data;
    
}
```

2. unknown 타입
- unknown 타입은 any 타입과 유사하게 모든 타입의 값이 할당될 수 있음
- 그러나 any를 제외한 다른 타입으로 선언된 변수에는 unknown 타입 값을 할당할 수 없음

| any                                 | unknown                              |
|-------------------------------------|--------------------------------------|
| 모든 타입의 값 할당 가능                      | 모든 타입의 값 할당 가능                       |
| any 타입은 어떤 타입으로도 할당 가능(단 never는 제외) | unknown 타입은 any 타입 외에 다른 타입으로 할당 불가능 |

```typescript
let unknownValue: unknown;

unknownValue = 123; // number
unknownValue = 'hello'; // string
unknownValue = () => {}; // function

let someValue1: any = unknownValue; // (O) any 타입에는 unknown 타입 할당 가능
let someValue2: number = unknownValue; // (X) number 타입에는 unknown 타입 할당 불가능
let someValue3: string = unknownValue; // (X) string 타입에는 unknown 타입 할당 불가능
```

**unknown 타입이 추가된 이유?**
- 함수를 unknown 타입 변수에 할당할때는 컴파일러가 아무런 경고를 주지 않지만 실행하면 에러가 발생
```typescript
// 할당하는 시점에서는 에러가 발생하지 않음
const unknownFunction: unknown = () => {};
unknownFunction(); // 에러 발생
```
- 함수뿐만 아니라 객체의 속성 접근, 클래스 생성자 호출을 통한 인스턴스 생성 등 객체 내부에 접근하는 모든 시도에서 에러 발생
- unknown 타입은 어떤 타입이 할당되었는지 알 수 없음을 나타내기 때문에 unknown 타입으로 선언된 변수는 값을 가져오거나 내부 속성에 접근할 수 없음
- 개발자가 임시로 사용했다가 수정을 깜박하는 등의 실수를 방지하기 위해 unknown 타입이 추가됨

3. void 타입
- 함수가 반환하는 값이 없을 때 사용
- 함수 내부에 별도 반환문이 없다면 ts 컴파일러가 void로 추론

4. never 타입
- 값을 반환할 수 없는 타입

**never 타입 사용 예시**
1. 에러를 던지는 경우
- 함수가 에러를 던지면 함수가 반환하는 값이 없기 때문에 never 타입으로 추론
```typescript
function generateError(res: Response): never {
    throw new Error(res.statusText);
}
```

2. 무한히 함수가 실행되는 경우
- 함수가 무한히 실행되면 함수가 반환하는 값이 없기 때문에 never 타입으로 추론
```typescript
function infiniteLoop(): never {
    while(true) {
        console.log('hello');
    }
}
```
- never 타입은 모든 타입의 하위 타입이다. 즉, never 자신을 제외한 어떤 타입도 never 타입에 할당할 수 없음
```typescript
let neverValue: never;
neverValue = 'hello'; // (X) never 타입에는 다른 타입 할당 불가능
```

5. Array 타입
- Object.prototype.toString.call() 메서드를 사용하여 배열을 구분할 수 있음
```typescript
const arr = [1, 2, 3];
console.log(Object.prototype.toString.call(arr)); // [object Array]
```

- 타입 스크립트에서는 정적 타입의 특성을 살려 명시적인 타입을 선언하여 해당 타입의 원소를 관리하는 것을 강제
```typescript
const arr: number[] = [1, 2, 3];
```

- 배열 타입을 명시하는 것만으로 배열의 길이까지는 제한할 수 없지만 튜플 타입을 사용하면 배열의 길이까지 제한할 수 있음
```typescript
let tuple: [number, string];
tuple = [1, '2']; // (O) 2개의 원소를 가진 튜플
tuple = [1, '2', 3]; // (X) 3개의 원소를 가진 튜플
```
- 예시 useState 훅
```typescript
const [count, setCount] = useState<number>(0);
```

- 튜플과 배열의 성질 혼합
```typescript
const httpStatusFromPaths: [number, string, ...string[]] = [
    200,
    'OK',
    'Created',
    'Accepted',
    'Non-Authoritative Information',
];

// 옵셔널 프로퍼티를 명시하고 싶다면 물음표 기호와 함께 선언
const optinalTuple1: [number, string?] = [1];
```

6. enum 타입
- 타입스크립트에서 지원하는 특수한 타입
```typescript
enum ProgrammingLanguage {
    Typescript, //0
    JavaScript, //1
    Python, //2
}

ProgrammingLanguage.Typescript; // 0
ProgrammingLanguage[1]; // JavaScript
```

- 각 멤버에 명시적으로 값 할당 가능
```typescript
enum ProgrammingLanguage {
    Typescript = 'Typescript',
    JavaScript = 'JavaScript',
    Java = 300,
    Python, // 301
    Rust //302
}
```

- enum 타입을 통해 응집력있는 집합 구조체를 만들 수 있으며, 사용자 입장에서도 간편하게 활용 가능
```typescript
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
        return false;
    case ItemStatusType.DELIVERED:
    default:
        return true;
}
};
```
- itemStatus 타입이 문자열로 지정된 경우와 비교한 장점 
  - 타입안전성: ItemStatusType에 명시되지 않은 다른 문자열은 인자로 받을 수 없어 타입 안전성이 우수함
  - 명확한 의미 전달과 높은 응집력: ItemStatusType이 다루는 값이 무엇인지 명확하고 아이템 상태에 대한 값을 모아놓은 것으로 응집력이 우수함
  - 가독성: 응집도가 높기 때문에 말하고자 하는 바가 더욱 명확함. 열거형 멤버를 통해 어떤 상태를 나타내는지 쉽게 알 수 있음
- 다만 열거형에 사용할 때는 주의해야 할 점
  - 숫자로만 이루어져 있거나 타입스크립트가 자동으로 추론한 열거형은 안전하지 않은 결과를 낳을 수 있음
  - 역방향으로도 접근할 수 있기 때문에 할당한 값을 넘어서는 범위로 접근하더라도 타입스크립트는 막지 않음
```typescript
ProgrammingLanguage[200]; // undefined -> 별다른 에러를 발생시키지 않음

const enum ProgrammingLanguage { 
//... 
}
```
- 다만, const enum으로 열거형을 선언하더라도 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 막지 못함
- 반면 문자열 상수로 관리되는 열거형은 의도하지 않은 값의 할당이나 접근을 방지하므로 문자열 상수로 관리하는 것이 도움이 됨

**열거형의 가장 큰 문제**
- 열거형은 타입공간과 값 공간에 모두 사용됨. 열거형은 즉시실행함수(IIFE) 형식으로 변환됨
- 이때 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생하여 불필요한 코드의 크기가 증가하는 결과를 초래할 수 있음
- 이러한 문제를 해결하기 위해 앞서 언급한 const enum 또는 as const assertion을 사용하여 유니온 타입으로 열거형과 동일한 효과를 얻는 방법이 있음
```typescript
const enum ProgrammingLanguage {
    Typescript = 'Typescript',
    JavaScript = 'JavaScript',
    Java = 'Java',
    Python = 'Python',
    Rust = 'Rust',
}

const languages = ["TypeScript", "JavaScript", "Python"] as const;
```

### 3.2 타입 조합
1. 교차 타입
- 두 개 이상의 타입을 조합하여 하나의 타입으로 만드는 방법
```typescript
type ProductItem = {
    id:number;
    name:string;
    price:number;
};

type ProductItemWidtDiscount = ProductItem & {discountAmount: number};
```

2. 유니온 타입
- 두 개 이상의 타입 중 하나의 타입만을 가질 수 있도록 만드는 방법
- 유니온 타입은 | 기호로 연결하여 사용
```typescript
type ProductItem = {
    id:number;
    name:string;
    price:number;
    quantity: number;
};

type CardItem = {
    id:number;
    name:string;
    type:string;
    imageUrl: string;
};

type PromotionEventItem = ProductItem | CardItem;

const printPromotionItem = () => {
    console.log(item.name) // O

    // quantity는 ProductItem에만 존재하는 속성이므로 컴파일 에러 발생
    console.log(item.quantity) // 컴파일 에러 발생
}
```

3. 인덱스 시그니처
- 특정 타입의 속성 이름은 알 수 없지만 속성값의 타입을 알고 있을 때 사용하는 문법
```typescript
interface IndexSignatureEx {
    [key: string]: number;
}
```
- 인덱스 시그니처를 선언할 때 다른 속성을 추가로 명시해줄 수 있는데 이때 추가로 명시한 속성은 인덱스 시그니처의 속성값과 타입이 일치해야 함
- 인덱스 시그니처의 속성값과 타입이 일치하지 않으면 에러 발생
```typescript
interface IndexSignatureEx2 {
    [key:string]: number | boolean;
    length: number;  // O
    isValid: boolean; // O
    name: string; // 에러 발생 -> 인덱스 시그니처의 키가 string일때는 number | boolean 타입만 허용 
}
```

4. 인덱스드 엑세스 타입
- 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용
```typescript
const PromotionList = [
    {type:"product", name:"chicken"},
    {type:"product", name:"pizza"},
    {type:"card", name:"cheer-up"},
];

type ElementOf<T> = typeof T[number];

// type PromotionItemType = { type: string; name: string; }
type PromotionItemType = ElementOf<PromotionList>;
```

5. 맵드 타입
- 다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법
```typescript
type Example = {
 a: number;
 b: string;
 c: boolean
}

type Subset<T> = {
    [K in keyof T]?:T[K];
}


const aExample:Subset<Example> = {a:3};
const bExample:Subset<Example> = {b:"hello"};
const cExample:Subset<Example> = {a:4, c:true};
```
- 맵드 타입에서는 수식어를 더하거나 제거해줄 수 있다
```typescript
type readOnlyEx = {
    readonly a: number;
    readonly b: string;
}

type createMutable<Type> = {
    -readonly [Property in keyof Type]: Type[Property];
}

type ResultType = createMutable<readOnlyEx>; // {a:number, b:string}

type OptionalEx = {
    a?: number;
    b?: string;
}

type Concrete<Type> = {
    [Property in keyof Type]-?: Type[Property];
}

type ResultType = Concrete<OptionalEx>; // {a:number, b:string}
```

**맵드 타입의 예시**
- BottomSheetStore라는 스토어를 만드는데 모든 바텀시트마다 일일이 스토어를 만들어준다면 코드의 중복이 발생 
```typescript


type BottomSheetStore = {
    RECENT_CONTACTS: {
        isOpen: boolean;
        isExpanded: boolean;
    };
    FAVORITE_CONTACTS: {
        isOpen: boolean;
        isExpanded: boolean;
    };
    ALL_CONTACTS: {
        isOpen: boolean;
        isExpanded: boolean;
    };
```
- 맵드 타입을 사용하면 코드의 중복을 줄일 수 있음
```typescript
const BottomSheetMap = {
    RECENT_CONTACTS: RecentContactsBottomSheet,
    FAVORITE_CONTACTS: FavoriteContactsBottomSheet,
    ALL_CONTACTS: AllContactsBottomSheet,
};

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

type BottomSheetStore = {
    [Key in BOTTOM_SHEET_ID]: {
        isOpen: boolean;
        isExpanded: boolean;
    };
};
```

- as 키워드를 사용해서 키를 재지정할수도 있음
```typescript
type BottomSheetStore = {
    [Key in BOTTOM_SHEET_ID as `${Key}BottomSheet`]: {
        isOpen: boolean;
        isExpanded: boolean;
    };
};
```

6. 템플릿 리터럴 타입
- 문자열 리터럴 타입을 조합하여 새로운 문자열 리터럴 타입을 만들 수 있는 문법
```typescript
type Stage = 'init' | 'loading' | 'success' | 'error';

type StageName = `${Stage}Stage`; // 'initStage' | 'loadingStage' | 'successStage' | 'errorStage'
```

7. 제네릭
- 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 외부에서 지정할 수 있도록 하는 문법
```typescript
type ExampleArrayType<T> = T[];

const array1: ExampleArrayType<number> = [1, 2, 3];
```

- 타입 추론이 가능한 경우에는 타입 명시를 생략할 수 있음
```typescript
function exampleFunc<T>(arg: T): T[] {
    return new Array(5).fill(arg);
}

const array1 = exampleFunc(1); // number[]
```

**제네릭을 사용할 때 주의할 점**
- 파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러 발생
- 제네릭의 꺾쇠괄호와 태그의 꺾쇠괄호를 혼동하여 문제가 생기는 것
- 제네릭 부분에 extends 키워드를 사용하여 컴파일러에게 특정 타입의 하위 타입만 올 수 있음을 알려주거나 function으로 선언
```typescript
// 에러 발생: JSX element 'T' has no corresponding closing tag
const arrowExmaple = <T>(arg: T): T[] => {
    return new Array(5).fill(arg);
}

// 정상 작동
const arrowExmaple = <T extends {}>(arg: T): T[] => {
    return new Array(5).fill(arg);
}
```

### 3.3 제네릭 사용법
1. 함수의 제네릭
- 어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용할 수 있음
```typescript
function ReadOnlyRepository<T>(target: ObjectType<T> | EntitySchema<T> | string): 
    Repository<T> {
        return getConnection('ro').getRepository(target);
}
```

2. 호출 시그니처의 제네릭
- 호출 시그니처: 타입스크립트의 함수 타입 문법으로 함수의 매개변수와 반환 타입을 미리 선언하는 것
- 호출 시그니처를 사용할 때 제네릭 타입을 어디에 위치시키는지에 따라 타입의 범위와 제네릭 타입을 언제 구체 타입으로 한정할지를 결정할 수 있음
```typescript
interface useSelectPaginationProps<T> {
    categoryAtom: RecoilState<number>;
    filterAtom: RecoilState<string[]>;
    sortAtom: RecoilState<SortType>;
    fetcherFunc: (props: CommonListRequest) => Promise<DefaultResponse<ContentListResponse<T>>>;
}
```

- 아래는 우아한형제들 배민선물하기 팀의 호출 시그니처 제네릭 활용 예시
```typescript
export type UseRequesterHookType = <RequestData = void, ResponseData = void>(
  baseURL?: 
      string | Headers,
  defaultHeader?: Headers
) => [RequestStatus, Requester<RequestData, ResponseData>];
```

3. 제네릭 클래스
- 제네릭 클래스는 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스
```typescript
class LocalDB<T> {
    // ...
    async put(table: string, row: T): Promise<T> {
      return new Promise<T>((resolved, rejected) = > { /* T 타입의 데이터를 DB에 저장 */ });
    }
  
    async get(table:string, key: any): Promise<T> {
      return new Promise<T>((resolved, rejected) = > { /* T 타입의 데이터를 DB에서 가져옴 */ });
    }
  
    async getTable(table: string): Promise<T[]> {
      return new Promise<T[]>((resolved, rejected) = > { /* T[] 타입의 데이터를 DB에서 가져옴*/ });
    }
  }
  
  export default class IndexedDB implements ICacheStore {
    private _DB?: LocalDB<{ key: string; value: Promise<Record<string, unknown>>;
    cacheTTL: number }>;
  
    private DB() {
      if (!this._DB) {
        this._DB = new LocalDB(“localCache”, { ver: 6, tables: [{ name: TABLE_NAME, keyPath: “key” }] });
      }
      return this._DB;
    }
    // ...
  }
```

4. 제한된 제네릭
- 타입 매개변수에 대한 제약 조건을 설정하는 기능
- 예를 들어 string 타입으로 제약하려면 타입 매개변수는 특정 타입을 상속해야 한다.
```typescript
type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never ? Partial<Record<Key, boolean>>: never;
```
- 이처럼 타입 매개변수가 특정 타입으로 묶였을 때 키를 바운드 타입 매개변수라고 부름. 그리고 string을 키의 상한 한계라고 함

5. 확장된 제네릭
- 제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수도 있음
```typescript
export class APIResponse<Ok, Err = string> {
  private readonly data: Ok | Err | null;
  private readonly status: ResponseStatus;
  private readonly statusCode: number | null;

  constructor(
    data: Ok | Err | null,
    statusCode: number | null,
    status: ResponseStatus
  ) {
    this.data = data;
    this.status = status;
    this.statusCode = statusCode;
  }

  public static Success<T, E = string>(data: T): APIResponse<T, E> {
    return new this<T, E>(data, 200, ResponseStatus.SUCCESS);
  }

  public static Error<T, E = unknown>(init: AxiosError): APIResponse<T, E> {
    if (!init.response) {
      return new this<T, E>(null, null, ResponseStatus.CLIENT_ERROR);
    }

    if (!init.response.data?.result) {
      return new this<T, E>(
        null,
        init.response.status,
        ResponseStatus.SERVER_ERROR
      );
    }

    return new this<T, E>(
      init.response.data.result,
      init.response.status,
      ResponseStatus.FAILURE
    );
  }

  // ...
}

// 사용하는 쪽 코드
const fetchShopStatus = async (): Promise<
  APIResponse<IShopResponse | null>
> => {
  // ...

  return (await API.get<IShopResponse | null>("/v1/main/shop", config)).map(
    (it) => it.result
  );
};
```

6. 제네릭 예시
- 현업에서 제네릭이 가장 많이 사용될 때는 API Response의 타입을 지정할 때
```typescript
export interface MobileApiResponse<Data> {
  data: Data;
  statusCode: string;
  statusMessage?: string;
}

export const fetchPriceInfo = (): Promise<MobileApiResponse<PriceInfo>> => {
    const priceUrl = "https: ~~~"; // url 주소

    return request({
        method: "GET",
        url: priceUrl,
    });
};

export const fetchOrderInfo = (): Promise<MobileApiResponse<Order>> => {
    const orderUrl = "https: ~~~"; // url 주소

    return request({
        method: "GET",
        url: orderUrl,
    });
};
```

**제네릭을 굳이 사용하지 않아도 되는 경우**
1. 타입이 다른 곳에서 사용되지 않을 때
```typescript
type GType<T> = T;
type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
interface Order {
  getRequirement(): GType<RequirementType>;
}
```
- GType은 다른 곳에서 사용되지 않고 getRequirement 함수의 반환 값 타입으로만 사용되고 있음
- GType이라는 이름이 현재 사용되는 목적을 정확히 담고 있지 않을뿐더러 코드를 읽는 사람이 혼란스러울 수 있음
- 아래처럼 사용하는 것이 낫다
```typescript
type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
interface Order {
  getRequirement(): RequirementType;
}
```

2. any 사용하기
- any 타입은 모든 타입을 허용하기 때문에 타입을 지정하는 의미가 사라짐
```typescript
type ReturnType<T = any> = {
    //...
}
```

3. 가독성을 고려하지 않은 사용
- 제네릭을 과하게 사용하면 코드의 가독성이 떨어질 수 있음
- 복잡한 제네릭은 의미 단위로 분할해서 사용하는 게 좋음

