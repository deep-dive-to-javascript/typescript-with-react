## 1. API 요청

### `fetch`로 API 요청하기

```tsx
const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0)

  useEffect(() => {
    fetch('https://api.baemin.com/cart').then(({ cartItem }) => {
      setCartCount(cartItem.length)
    })
  }, [])

  // cartCount 상태를 이용하여 컴포넌트 렌더링
}
```

- 백엔드 기능 변경을 위해 API URL을 수정할 경우
  - 변경 요구에 취약: 이미 컴포넌트 내부에 깊숙이 자리 잡은 비동기 호출 코드
  - 새로운 API 요청 정책 추가 시마다 수정해야 하는 번거로움
    - “여러 서버에 API를 요청할 때 타임아웃 설정이 필요하다”
    - “모든 요청에 커스텀 헤더가 필요해요”

### 서비스 레이어로 분리하기

> 비동기 호출 코드는 컴포넌트 영역에서 분리되어 다른 영역(서비스 레이어)에서 처리되어야 한다.

- `fetch` 함수를 호출하는 부분이 서비스 레이어로 이동
- 컴포넌트가 서비스 레이어 비동기 함수를 호출하고 그 결과를 받아와 렌더링

```tsx
async function fetchCart() {
  const controller = new AbortController()

  const timeoutId = setTimeout(() => controller.abort(), 5000)

  return await fetch('https://api.baemin.com/cart', {
    signal: controller.signal,
  })
}
```

> 하지만 단순히 `fetch` 함수를 분리한다고 다양한 API 정책이 추가되는 문제를 해결하기 어렵다.

### Axios 활용하기

```tsx
const apiRequester: AxiosInstance = axios.create({
  baseURL: 'https://api.baemin.com',
  timeout: 5000,
})

const fetchCart = (): AxiosPromise<FetchCartResponse> => apiRequester.get<FetchCartResponse>('cart')

const postCart = (postCartRequest: PostCartRequest): AxiosPromise<PostCartResponse> =>
  apiRequester.post<PostCartRequest>('cart', postCartRequest)
```

#### API Entry가 여러 개인 경우

- API Entry가 2개 이상일 경우에는 각 서버의 기본 URL을 호출하도록 API 요청 처리 인스턴스를 따로 구성
- 이후 서비스 코드 호출 시 각각의 apiRequester를 사용

```tsx
const apiRequester: AxiosInstance = axios.create(defaultConfig)
const orderApiRequester: AxiosInstance = axios.create({
  baseURL: 'https://api.baemin.order/',
  ...defaultConfig,
})

const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: 'https://cart.baemin.order/',
  ...defaultConfig,
})
```

### Axios 인터셉터 사용하기

- requester별로 다른 헤더를 설정해줘야 하는 경우 Axios의 인터셉터 기능을 사용하여 requester에 따라 비동기 호출 내용을 추가해서 처리할 수 있음
- 또한 API 에러를 처리할 때 하나의 에러 객체를 묶어서 처리할 수도 있음

#### `apiRequester.interseptors`

- `interceptors` 로 header를 설정하는 기능 추가하기

```tsx
const apiRequester: AxiosInstance = axios.create({
  baseURL: 'https://api.baemin.com',
  timeout: 5000,
})

const setRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = requestConfig
  config.headers = {
    ...config.headers,
    'Content-Type': 'application/json;charset=utf-8',
    user: getUserToken(),
    agent: getAgent(),
  }
  return config
}

apiRequester.interceptors.request.use(setRequestDefaultHeader)
```

#### `orderApiRequester.interseptors`

- 기본 `apiRequester`와 다른 header 설정하는 기능 및 에러 처리

```tsx
const setOrderRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = requestConfig
  config.headers = {
    ...config.headers,
    'Content-Type': 'application/json;charset=utf-8',
    'order-client': getOrderClientToken(),
  }
  return config
}

const orderApiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
})

orderApiRequester.interceptors.request.use(setOrderRequestDefaultHeader)
orderApiRequester.interceptors.response.use((response: AxiosResponse) => response, httpErrorHandler)
```

#### 빌더 패턴

- 요청 옵션에 따라 다른 인터셉터를 만들기 위해 빌더 패턴을 추가하여 `APIBuilder` 같은 **클래스** 형태로 구성하기도 함
- 구성 형태
  - 기본 `API` 클래스: 실체 호출 부분을 구성
  - 빌더 패턴 `APIBuilder` 클래스: API 호출하기 위한 wrapper 형태로 구성
  - 필요한 요청 옵션을 메서드 형태로 작성하여 사용
- 장점과 단점
  - 장점: 옵션이 다양한 경우에 인터셉터를 설정값에 따라 적용하고, 필요 없는 인터셉터를 선택적으로 사용할 수 있음
  - 단점: 보일러플레이트 코드가 많아짐

```tsx
type Response<T> = { data: T }
type JobNameListResponse = string[]

const fetchJobNameList = async (name?: string, size?: number) => {
  const api = APIBuilder.get('/apis/web/jobs').withCredentials(true).params({ name, size }).build()
  const { data } = await api.call<Response<JobNameListResponse>>()
  return data
}
```

### API 응답 타입 지정하기

- API의 응답 값을 하나의 Response 타입 지정 → 같은 서버에서 오는 응답의 형태는 대체로 통일이 가능

```tsx
interface Response<T> {
  data: T
  status: string
  serverDateTime: string
  errorCode?: string
  errorMessage?: string
}
```

- 응답이 있는 `GET` / `POST`에 사용하기

```tsx
const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> =>
  apiRequester.get<Response<FetchCartResponse>>('cart')

const postCart = (postCartRequest: PostCartRequest): AxiosPromise<Response<PostCartResponse>> =>
  apiRequester.post<Response<PostCartResponse>>('cart', postCartRequest)
```

#### 응답이 없을 수 있는 `UPDATE` / `CREATE`

- `UPDATE`, `CREATE` 같이 응답이 없을 수 있는 API를 처리하기 까다로움

```tsx
const updateCart = (updateCartRequest): AxiosPromise<Response<FetchCartResponse>> =>
  apiRequseter.get<null>('cart')
```

> 따라서, Response 타입은 `apiRequester`가 모르게 관리되어야 한다.

#### 서버에서 서버로 API 요청 및 응답 값을 넘겨주기만 하는 경우 타입 정의하기

- 해당 값에 어떤 응답이 들어있는지 알 수 없거나 값의 형식이 달라지더라도 로직에 영향을 주지 않는 경우에는 `unknown` 타입을 사용하여 알 수 없는 값임을 표현해야 함

```tsx
interface response {
  data: {
    cartItems: CartItem[]
    forPass: unknown
  }
}
```

- 만약 `forPass` 안에 프론트 로직에서 사용해야 하는 값이 있다면 unknown을 유지하되,
- 프론트 로직에서 써야 하는 값에 대해서만 타입을 선언한 다음에 사용

```tsx
type forPass {
	type: "A" | "B" | "C"
}

const isTargetValue = () => (data.forPass as ForPass).type === "A"
```

### 뷰 모델(View Model) 사용하기

- API 변경 가능성 대응
- 뷰 모델을 사용하여 API 변경에 따른 범위 한정

#### 특정 객체 리스트를 조회하는 `fetchList` API

```tsx
interface ListResponse {
  items: ListItem[]
}

const fetchList = async (filter?: ListFetchFilter): Promise<ListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get('/apis/get-list-summarties')
    .call<Response<ListResponse>>()

  return { data }
}

const ListPage: React.FC = () => {
  const [totalItemCount, setTotalItemCount] = useState(0)
  const [items, setItems] = useState<ListItem[]>([])

  useEffect(() => {
    fetchList(filter).then(({ itmes }) => {
      setCartCount(items.length)
      setItems(items)
    })
  }, [])

  return (
    <div>
      <Chip label={totalItemCount} />
      <Table items={items} />
    </div>
  )
}
```

#### 뷰 모델 사용하기

- 뷰 모델을 사용하면 API 응답이 변경되더라도 UI가 깨지지 않게 개발 가능
- API 응답에 없는 도메인 개념의 값들도 백엔드, UI 로직 추가 없이 필드 추가를 통해 사용 가능

```tsx
interface JobListItemResponse {
  name: string
}

interface JobListResponse {
  jobItems: JobListItemResponse[]
}

class JobList {
  readonly totalItemCount: number
  readonly items: JobListItemResponse[]
  constructor({ jobItems }: JobListResponse) {
    this.totalItemCount = jobItems.length
    this.items = jobItems
  }
}

const fetchJobList = async (filter?: ListFetchFilter): Promise<JobListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get('/apis/get-list-summaries')
    .call<Response<JobListResponse>>()

  return new JobList(data)
}
```

#### 뷰 모델의 단점 및 주의 사항

- 뷰 모델은 서버에서 지정한 응답 형식에 맞춰 구성되었기 때문에 UI에서 사용하려면 더 많은 타입을 선언할 수밖에 없음
- 뷰 모델 사용 시 문제점
  - 추상화 레이어 추가로 인한 복잡한 코드 → 코드 가독성 저하 문제
  - 레이어를 관리하고 개발해야 함 → 값비싼 비용 문제
  - 서버가 내려준 응답과 클라이언트가 실제 사용하는 도메인이 상이해질 수 있음 → 서버와 클라이언트 간의 의사소통 문제
- 뷰 모델 사용 시 주의 사항
  - 꼭 필요한 곳에만 뷰 모델을 부분적으로 만들어서 사용하기
  - 백엔드와 클라이언트 개발자 간의 충분한 소통으로 API 응답 변화 최대한 줄이기
  - 뷰 모델에 필드를 추가하는 대신에 `getter` 등의 함수로 사용하기

### Superstruct를 사용해 런타임에서 응답 타입 검증하기

> [!NOTE]
>
> #### Superstruct
>
> - zod와 비슷한 쓰임새의 라이브러리
> - 인터페이스 정의와 자바스크립트 데이터의 유효성 검사
> - 런타임에서의 데이터 유효성 검사를 통해 개발자와 사용자에게 자세한 런타임 에러 제공

#### 주요 모듈 확인하기

```tsx
import { assert, is, validate, object, number, string, array } from 'superstruct'

const Article = object({
  id: number(),
  title: string(),
  tags: array(string()),
  author: object({
    id: number(),
  }),
})

const data = {
  id: 34,
  title: 'Hello World',
  tags: ['news', 'features'],
  author: {
    id: 1,
  },
}

assert(data, Article)
is(data, Article)
validate(data, Article)
```

- `assert`: 유효하지 않을 경우 에러를 던짐
- `is`: 유효성 검사 결과에 따라 boolean 값 반환
- `validate`: `[error, data]` 형식의 튜플 반환
  - 유효하지 않을 경우 에러 값 반환
  - 유효한 경우 첫 번째 요소로 `undefined`, 두 번째 요소로 `data` 반환

## 2. API 상태 관리하기

- 실제 API를 요청하는 코드는 컴포넌트 내에서 비동기 함수를 직접 호출하지 않음
- 비동기 API를 호출하기 위해서는 API 성공 및 실패에 따른 **상태가 관리되어야 함**
- 이를 위해 상태 관리 라이브러리의 **액션**이나 **훅**과 같이 재정의된 형태를 사용해야 함

### 상태 관리 라이브러리에서 호출하기

- 상태 관리 라이브러리의 비동기 함수들은 서비스 코드를 사용해서 비동기 상태를 변화시킬 수 있는 함수 제공
- 컴포넌트는 이러한 함수를 사용해서 상태 구독 및 상태 변경 시 컴포넌트 리렌더링
- 상태 관리 라이브러리
  - Redux:
    - 전역 상태 관리 라이브러리로, 비동기 상태 관리를 위해선 미들웨어를 도입해야 함
    - 많은 보일러플레이트
  - MobX
    - Redux의 단점을 개선한 비동기 상태 관리 라이브러리
    - 비동기 콜백 함수를 분리한 액션
    - `runInAction`과 같은 상태 변경 처리 메서드
    - `async`/`await` 및 `flow` 등의 비동기 상태 관리를 위한 기능 추가
- 모든 상태 관리 라이브러리의 문제점
  - 비동기 처리 함수를 호출하기 위해 액션이 추가될 때마다 관련된 스토어나 상태가 늘어남
  - 전역 상태 관리자가 모든 비동기 상태에 접근하고 변경 가능
  - 쓸데없는 비동기 통신이 발생하거나 의도치 않은 상태 변경 발생 가능

### 훅으로 호출하기

- 캐시를 사용하여 비동기 함수를 호출
- 상태 관리 라이브러리에서 발생했던 의도치 않은 상태 변경을 방지

#### react-query를 사용한 비동기 처리 커스텀 훅

- Job 목록을 불러오는 `useFetchJobList`

```tsx
const useFetchJobList = () => {
  return useQuery(['fetchJobList'], async () => {
    const response = await JobService.fetchJobList()

    return new JobList(response)
  })
}
```

- Job 1개를 업데이트 하는 `useUpdateJob`

```tsx
const useUpdateJob = (
  id: number,
  { onSuccess, ...options }: UseMutationOptions<void, Error, JobUpdateFormValue>,
): UseMutationResult<void, Error, JobUpdateFormValue> => {
  const queryClient = useQueryClient()

  return useMutation(
    ['updateJob', id],
    async (jobUpdateForm: JobUpdateFormValue) => {
      await JobService.updateJob(id, jobUpdateForm)
    },
    {
      onSuccess: (
        data: void, // status 200으로만 성공 판별
        values: JobUpdateFormValue,
        context: unknown,
      ) => {
        // 성공 시 ‘fetchJobList’를 유효하지 않음으로 설정
        queryClient.invalidateQueries(['fetchJobList'])

        onSuccess && onSuccess(data, values, context)
      },
      ...options,
    },
  )
}
```

#### 폴링을 사용하여 최신 상태 업데이트하기

> [!NOTE]
>
> #### 폴링(polling)
>
> - 클라이언트가 주기적으로 서버에 요청을 보내 데이터를 업데이트하는 것
> - 클라이언트는 일정한 시간 간격으로 서버에 요청을 보내고, 서버는 해당 요청에 대해 최신 상태의 데이터 응답

```tsx
const JobList: React.FC = () => {
  const { isLoading, isError, error, refetch, data: jobList } = useFetchJobList()

  // 30초 간격으로 갱신하는 간단한 Polling 로직
  useInterval(() => refetch(), 30000)

  if (isLoading) return <LoadingSpinner />

  if (isError) return <ErrorAlert error={error} />

  return (
    <>
      {jobList.map(job => (
        <Job job={job} />
      ))}
    </>
  )
}
```

#### 비동기 통신을 react-query를 사용해서 처리하는 이유

- 에러 발생, 로딩 중과 같은 상태는 전역으로 관리할 필요가 없음
- 필요하지 않은 다른 컴포넌트의 상태를 구독은 컴포넌트의 높은 결합도와 복잡도로 이어짐 → 유지보수의 어려움 발생

## 3. API 에러 핸들링

- 비동기 API 호출을 하다 보면 상태 코드에 따라 다양한 에러 발생 가능
  - `401`(인증되지 않은 사용자)
  - `404`(존재하지 않는 리소스)
  - `500`(서버 내부 에러)
  - `CORS` 에러
- 적절한 에러 핸들링의 장점
  - 에러 상황에 대한 명시적인 코드로 유지보수 용이
  - 사용자에게 구체적인 에러 상황 전달 가능

### 타입 가드 활용하기

#### 서버에서 전달하는 공통 에러 객체에 대한 타입 정의

```tsx
interface ErrorResponse {
  status: string
  serverDateTime: string
  errorCode: string
  errorMessage: string
}
```

#### 처리해야 할 Axios 에러 형태에 대한 타입 가드

- 사용자 정의 타입 가드로 서버 에러를 명시적으로 확인
- `is` 연산자를 사용하여 반환 타입을 명시적으로 작성

```tsx
function isServerError(error: unknown): error is AxiosError<ErrorResponse> {
  return axios.isAxiosError(error)
}

const onClickDeleteHistoryButton = async (id: string) => {
  try {
    await axios.post('https://....', { id })

    alert('주문 내역이 삭제되었습니다.')
  } catch (error: unknown) {
    if (isServerError(e) && e.response && e.response.data.errorMessage) {
      setErrorMessage(e.response.data.errorMessage)
      return
    }

    setErrorMessage('일시적인 에러가 발생했습니다. 잠시 후 다시 시도해주세요')
  }
}
```

### 에러 서브클래싱하기

- 서브클래싱을 활용하여 실제 요청을 처리할 때 단순한 서버 에러 외에 발생하는 다양한 에러를 명시적으로 표시

> [!NOTE]
>
> #### 서브클래싱(Subclassing)
>
> - 기존(상위 또는 부모) 클래스를 확장하여 새로운(하위 또는 자식) 클래스를 만드는 과정
> - 새로운 클래스는 상위 클래스의 모든 속성과 메서드를 상속받아 사용 가능
> - 추가적인 속성 및 메서드 정의 가능

```tsx
const getOrderHistory = async (page: number): Promise<History> => {
  try {
    const { data } = await axios.get(`https://some.site?page=${page}`)
    const history = await JSON.parse(data)

    return history
  } catch (error) {
    alert(error.message)
  }
}
```

- 개발자 입장에서 어떤 에러가 발생되었는지 구분할 수 없음
- 서브클래싱을 활용
  - 에러 발생 시 코드상에서 어떤 에러인지 바로 확인 가능
  - 에러 인스턴스에 따라 에러 처리 방식 다르게 구현 가능 → `instanceof` 연산자 사용

```tsx
const alert = (meesage: string, { onClose }: { onClose?: () => void }) => {}

const onActionError = (error: unknown, params?: Omit<AlertPopup, 'type' | 'message'>) => {
  if (error instanceof UnauthorizedError) {
    onUnauthorizedError(error.message, errorCallback?.onUnauthorizedErrorCallback)
  } else if (error instanceof NetworkError) {
    alert('네트워크 연결이 원활하지 않습니다. 잠시 후 다시 시도해주세요.', {
      onClose: errorCallback?.onNetworkErrorCallback,
    })
  } else if (error instanceof OrderHttpError) {
    alert(error.message, params)
  } else if (error instanceof Error) {
    alert(error.message, params)
  } else {
    alert(defaultHttpErrorMessage, params)
  }

  const getOrderHistory = async (page: number): Promise<History> => {
    try {
      const { data } = await fetchOrderHistory({ page })
      const history = await JSON.parse(data)

      return history
    } catch (error) {
      onActionError(error)
    }
  }
}
```

### 인터셉터를 활용한 에러 처리

- Axios 같은 페칭 라이브러리에서 제공하는 기능
- HTTP 에러에 일관ㄹ된 로직 적용 가능

```tsx
const httpErrorHandler = (error: AxiosError<ErrorResponse> | Error): Promise<Error> => {
  error => {
    // 401 에러인 경우 로그인 페이지로 이동
    if (error.response && error.response.status === 401) {
      window.location.href = `${backOfficeAuthHost}/login?targetUrl=${window.location.href}`
    }
    return Promise.reject(error)
  }
}

orderApiRequester.interceptors.response.use((response: AxiosResponse) => response, httpErrorHandler)
```

### 에러 바운더리를 활용한 에러 처리

- 리액트 컴포넌트 트리에서 에러가 발생할 때 공통으로 에러를 처리하는 리액트 컴포넌트
- 리액트 컴포넌트 트리 하위에 있는 컴포넌트에서 발생한 에러 캐치 가능
- 해당 에러를 가장 가까운 부모 에러 바운더리에서 처리 가능
- 에러가 발생한 컴포넌트 대신에 에러 처리를 하거나 예상치 못한 에러를 공통 처리하는 경우에 사용

### react-query를 활용한 에러 처리

```tsx
const JobComponent: React.FC = () => {
  const { isError, error, isLoading, data } = useFetchJobList()

  if (isError) <div>{`${error.message}가 발생했습니다. 나중에 다시 시도해주세요.`}</div>
  if (isLoading) <div>로딩 중입니다.</div>

  return (
    <>
      {data.map(job => (
        <JobItem key={job.id} job={job} />
      ))}
    </>
  )
}
```

### 커스텀 에러 처리

- API 응답은 주로 성공 시 `2xx` 코드, 실패 시 `4xx`, `5xx` 코드 반환
- 비즈니스 로직에서의 유효성 검증에 의해 추가된 커스텀 에러의 경우 `200` 응답과 함께 응답 바디에 별도의 사태 코드를 전달하기도 함

#### 장바구니에서 주문 생성 API가 반환하는 커스텀 에러

```text
httpStatus: 200
{
	"status": "C20005", // 성공인 경우 "SUCCESS"를 응답
	"message": "장바구니에 품절된 메뉴가 있습니다."
}
```

#### 조건문으로 `status`(상태) 비교

```tsx
const successHandler = (response: CreateOrderResponse) => {
  if (response.status === 'SUCCESS') {
    // 성공 시 진행할 로직...
    return
  }
  throw new CustomError(response.status, response.message)
}
const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post('https://...', data)

    successHandler(response)
  } catch (error) {
    errorHandler(error)
  }
}
```

- 영향 범위가 각 요청에 대한 성공/실패 응답 처리 함수로 한정되어 관리에 용이
- 이런 방식으로 처리해야 하는 API가 많을 때 매번 구문 추가가 필요하다는 문제

#### 커스텀 에러를 사용하고 있는 요청에 대한 일괄적인 에러 처리하기

- Axios 등의 라이브러리 인터셉터 기능 활용
- 특정 호스트에 대한 API requester 별도 선언 → 상태 코드 비교 로직으로 인터셉터에 추가

```tsx
export const apiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
})

export const httpSuccessHandler = (response: AxiosResponse) => {
  if (response.data.status !== 'SUCCESS') {
    throw new CustomError(response?.data.message, response)
  }

  return response
}

apiRequester.interceptors.response.use(httpSuccessHandler, httpErrorHandler)

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post('https://...', data)

    successHandler(response)
  } catch (error) {
    // status가 SUCCESS가 아닌 경우 에러로 전달
    errorHandler(error)
  }
}
```
