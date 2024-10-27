# 7장 비동기 호출

2024.10.27 dasom



## 7.1 API 요청

### 1. fetch로 API 요청하기

장바구니 내역 조회 및 내역 아이템 갯수 표시를 위한 api 

```ts
const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0)
  
  useEffect(() => {
    fetch("https://api.baemin.com/cart")
    .then((res) => res.json())
    .then(({cartIem}) => {
      setCartCOunt(cartItem.length)
    })
  }, [])
  
  // 이하 생략
}
```

한계점

* 컴포넌트 내부에 자리 잡은 비도익 호출 코드는 변경 요구에 취약함
* API 정책이 바뀌거나 추가될 때마다 매 코드를 수정해야하는 번거로움이 발생함



### 2. 서비스 레이어로 분리하기

코드 변경 요구에 취약하다는 한계점을 극복하기 위해서는 -> 비동기 호출 코드가 컴포넌트 영역에서 분리되어 다른 영역(서비스 레이어)에서 처리되어야 함

하지만 fetch 함수만을 분리하는 것으로는 API 정책 변경 (e.g. 매 요청마다 timeout이 필요하다) 등에 대해서는 여전히 번거로움이 존재함



### 3. Axios 활용하기

fetch는 내장 라이브러리이기 때문에 많은 기능을 사용하기 위해서는 직접 구형해야함. 

이를 해결하기 위해 Axios 라이브러리를 주로 사용한다.

```ts
const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000,
}) // API Entry가 여러개일 경우 이를 여러개 생성해서 처리 가능

const fetchCart = (): AxiosPromise<FetchCartResponse> => 
	apiRequester.get <FetchCartResponse>("cart");

const postCart = (postCartRequest: PostCartRequest): AxiosPromise<FetchCartResponse> => 
	apiRequester.post <posthCartResponse>("cart", PostCartRequest);
```



### 4. Axios 인터텝터 활용하기

requester에 따라 다른 헤더를 설정해야할 수도 있다.

```ts
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: orderCartApiBaseUrl,
  ...defaultConfig,
})
orderCartApiRequester.interceptors.request.use(SetRequestDefaultHeader);
```

cf. 요청 옵션에 따라 다른 인터셉터를 만들기 위해서는 빌더 패턴을 추가하여 APIBuilder와 같은 클래스 형태로 구성하기도 한다.



### 5. API 응답 타입 지정하기

같은 서버에서 오는 응답의 형태는 대체로 통일되어 있어 하나의 Response 타입으로 묶일 수 있다. 하지만 Response타입을 api-requester 내에서 처리하지 않도록 주의해야 한다. 

또한 해당 값에 어떤 응답이 들어있는지 알 수 없거나 값의 형식이 달라지더라도 로직에 영향을 주지 않는 경우에는 unknown 타입을 사용하여 알 수 없는 값임을 표시 할 수 있다.



### 6. 뷰 모델(View Model) 사용하기

API 응답은 변할 가능성이 커, 뷰 모델을 통해 API 변경에 따른 범위를 한정할 수 있다.

> 더 쉬운 이해를 위한 예시 코드 (from Chat GPT🤖)

```js
// API 응답 예시
const apiResponse = {
  first_name: "John",
  last_name: "Doe",
  age: 25,
  created_at: "2024-10-27T10:00:00Z"
};

// 뷰 모델 생성
const createUserViewModel = (response) => {
  return {
    fullName: `${response.first_name} ${response.last_name}`,
    age: response.age,
    registeredDate: new Date(response.created_at).toLocaleDateString()
  };
};

// UI에서 사용하는 뷰 모델
const userViewModel = createUserViewModel(apiResponse);

console.log(userViewModel);
// 출력: { fullName: "John Doe", age: 25, registeredDate: "10/27/2024" }

```

하지만 매 응답에 대해 뷰모델을 만들게 되면 역시나 교체 비용이 많이 드는 작업이 된다.

따라서 꼭 필요한 곳에만 뷰 모델을 부분적으로 만들어서 사용하고, API 응답 변화를 최대한 줄이는 식으로 문제를 개선할 수 있다.



### 7. Superstruct를 사용해 런타입에서 응답 타입 검증하기

>  Superstruct란?
>
> 런타임 응답 타입 검증을 하기 위해 사용하는 라이브러리

```ts
const Article = object({
  id: number(),
  title: string()
})

const data = {
  id: 34,
  title: "Hello"
}

assert(data, Article) // 유효하지 않을 경우 에러를 던진다
is(data, Article) // is 유효성 검사 결과에 따라 boolean을 반환한다
validate(data, Article) // validate [error, data] 형식의 튜플을 반환한다. 유효하지 않을 때는 에러 값이 반환되고 유효한 경우에는 첫번째 요소로 undefined, 두번째 요소로 data value가 반환된다.
```



## 7.2 API 상태 관리하기

### 1. 상태 라이브러리에서 호출하기

Redux

* 초기에 나온 상태 관리 라이브러리
* 비동기 상태가 아닌 전역 상태를 위해 만들어진 라이브러리로, 비동기 상태 관리를 위해 미들웨어 도입이 필요함
* 따라서 보일러플레이트 코드가 많아지는 단점이 존재함

MobX

* 비동기 콜백함수를 분리하는 등 비동기 상태 관리를 위한 기능이 존재함

모든 상태 관리 라이브러리에서의 공통적 문제점

* 전역 상태 관리자가 모든 비동기 상태에 접근하고 변경할 수 있다는 점

  (만약 2개 이상 컴포넌트가 구독하고 있는 비동기 상태가 있다면 의도치 않은 상태변경이 발생할 수도 있다.)



### 2. 훅으로 호출하기

react-query나 useSwr 같은 훅을 사용하는 방법이 상태 변경 라이브러리를 사용하는 것 보다 훨씬 간단한 편이다.

이러한 훅을 사용하면서, 컴포넌트가 반드시 최신 상태를 표현하기 위해서는 폴링이나 웹소켓 등의 방법을 사용해야 한다.

> cf. 폴링(Polling)이란?
>
> 클라이언트가 주기적으로(일정 간격으로) 서버에 요청을 보내 데이터를 업데이트 하는 것이다.



## 7.3 API 에러 핸들링

비동기 API 호출을 하다 보면 상태 코드에 따라 401(인증되지 않은 사용자), 404(존재하지 않는 리소스), 500 (서버 내부 에러), CORS 에러 등 다양한 에러가 발생할 수 있다.

### 1. 타입 가드 활용하기

Axios라이브러리는 isAxiosError라는 타입 가드를 제공하고 있지만, 서버 에러임을 명확하게 표시하기 위해 타입 가드를 별도로 활용할 수 있다.



### 2. 에러 서브클래싱하기

단순 서버 에러 뿐 아니라 인증 전보 에러, 네트워크 에러 등을 명시적으로 표시하기 위해 서브클래싱을 활용할 수 있다.

> cf. 서브클래싱?
>
> 기존 클래스를 확장하여 새로운 (하위) 클래스를 만드는 과정을 말한다.

Axios를 사용하고 있다면 조건에 따라 인터셉터에서 적합한 에러 객체를 전달하는 것이 가능하다.



### 3. 인터셉터를 활용한 에러 처리

Axios 같은 페칭 라이브러리는 인터셉터 기능을 제공한다. 이를 통해 HTTP 에러에 일관된 로직을 적용할 수 있다.



### 4. 에러 바운더리를 활용한 에러 처리

에러 바운더리는 리액트 컴포넌트 트리에서 에러가 발생할 때 공통으로 에러를 처리하는 리액트 컴포넌트이다. 이를 사용해 에러가 발생한 컴포넌트 대신 에러를 ㄹ처리하거나 예상치 못한 에러를 공통 처리할 때 사용할 수 있다.



### 5. 상태 관리 라이브러리에서의 에러 처리

비동기 처리 시 에러 상태를 저장해 컴포넌트에서 해당 값을 활용할 수 있다.



### 6. React-query를 활용한 에러 처리

```ts
const JobComponent = () => {
  const{ isError, error, isLoading, data } = useFetchJobList();
  
  if (isError){
    return <div>{`${error.message}가 발생했습니다.`}</div>
  }
  
  if (isLoading) {
      return <div>로딩 중...</div>
  }
  
  return <>{data.map((job) => <JobItem key={job.id} job={job}/>)}</>;
}
```



### 7. 그 밖의 에러 처리

status 200과 함께 응답 바디에 별도의 상태 코드를 전달하기도 한다.

```
httpStatus:200
{
	"status": "C101",
	"message": "품절"
}
```

이런 경우 if 문으로 커스텀된 status를 구분하며, 별도의 인터셉터를 추가해 구분할 수도 있다.



## 7.4 API 모킹

서버 API가 완성되기 전에 개발을 진행해야하는 경우, mock API를 통해 개발을 진행할 수 있다. (mocking: 가짜 모듈을 활용하는 것)

### 1. JSON 파일 불러오기

간단한 조회만 필요한 경우에는 json 파일을 만들어 import 해서 테스트할 수 있다. 이 방법은  환경설정이 별도로 필요하지 않다는 장점이 있다.



### 2. NextApiHandler 활용하기

프로젝트에서 Next.js를 사용하고 있다면 활용할 수 있다.

하나의 파일 안에 하나의 핸들러를 default export로 구현해야하며 파일 경로가 요청 경로가 된다.



### 3. API 요청 핸들러에 분기 추가하기

개발이 완료된 이후에도 필요에 따라 목업 함수를 사용할 수 있다는 장점이 있지만 모든 API 요청에 if 분기문을 추가해야해 번거롭게 느껴질 수 있다.



### 4. axios-mock-adapter로 모킹하기

axios-mock-adapter는 Axios 요청을 가로채서 요청에 대한 응답 값을 대신 반환한다. 또한 앞선 두가지 방법과 달리 mock API의 주소가 필요하지 않다.



### 5. 목업 사용 여부 제어하기

로컬에서는 목업을 사용하고 dev나 운영 환경에서는 사용하지 않으려면 간단히 플래그를 설정해 제어할 수 있다.

package.json에 별도의 start 명령어를 만들 수도 있고, env에 지정된 변수로 구분할 수도 있다.





