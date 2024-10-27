# 비동기 호출

<aside>
💡

프론트엔드와 백엔드 통신 시, API로 요청하고 응답하는 모든 행위가 비동기 처리된다.

비동기 처리 관련 고려사항

- 비동기 동작이 어떤 상태인지
- 비동기 동작을 위해 필요한 정보
- 요청 성공 시, 정보 저장 관리
- 요청 실패 시, 실패 정보 확인
- 비동기 요청 코드 관리
</aside>

## API 요청

![배지 (badge) : 사용자에게 새로운 것이 있음을 알려주고자 사용](https://github.com/user-attachments/assets/5515a458-c2b2-42b7-8e55-c2ca605fdc0a)

배지 (badge) : 사용자에게 새로운 것이 있음을 알려주고자 사용

1. fetch 로 API 요청
    1. 장점 : 내장 기능으로 별도의 설치 필요없음
    2. 단점 : API 수정 시, 코드 수정 번거로움
2. 서비스 레이어로 분리하기 : 비동기 호출 코드를 컴포넌트 영역에서 분리해서 다른 영역에서 처리
    1. 장점 : 코드 분리를 통해 재사용성, 유지보수성, 일관성 향상 (코드 수정 범위를 줄여줌)
    2. 단점 : API 수정 번거로움의 완전한 해결책이 아님
3. Axios 활용하기
    1. 장점 : 많은 기능들을 제공하기에 직접 구현할 필요 없음

### Axios vs fetch

```jsx
//커스텀 헤더
function fetchWithCustomHeaders(url, options = {}) {
  const defaultHeaders = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${localStorage.getItem('token')}`,
  };

  return fetch(url, {
    ...options,
    headers: { ...defaultHeaders, ...options.headers },
  });
}

// 사용 예시
fetchWithCustomHeaders('/api/some-endpoint')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error("Error:", error));

//쿠키
function fetchWithCredentials(url, options = {}) {
  return fetch(url, {
    ...options,
    credentials: 'include', // same-origin 또는 include로 설정
  });
}

// 사용 예시
fetchWithCredentials('/api/some-endpoint')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error("Error:", error));

//토큰
async function fetchWithAuth(url, options = {}) {
  const response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${localStorage.getItem('token')}`
    },
  });

  if (response.status === 401) { // 토큰 만료 시
    const newToken = await refreshAuthToken(); // 새 토큰 발급 함수 호출
    localStorage.setItem('token', newToken);
    // 새 토큰으로 다시 요청
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${newToken}`
      },
    });
  }

  return response;
}

async function refreshAuthToken() {
  const response = await fetch('/auth/refresh', { method: 'POST' });
  const data = await response.json();
  return data.newToken; // 예시: 새로 발급된 토큰 반환
}

// 사용 예시
fetchWithAuth('/api/some-endpoint')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error("Error:", error));

//엔트리 2개
const API_BASE_URLS = {
  entry1: 'https://api.baemin.or/',
  entry2: 'https://cart.baemin.order/'
};

function fetchFromEntry(entry, endpoint, options = {}) {
  const baseURL = API_BASE_URLS[entry];
  if (!baseURL) throw new Error(`Invalid API entry: ${entry}`);
  
  return fetch(`${baseURL}${endpoint}`, options)
    .then(response => {
      if (!response.ok) throw new Error("Network response was not ok");
      return response.json();
    });
}

// 사용 예시
fetchFromEntry('entry1', '/users')
  .then(data => console.log("Entry 1 Data:", data))
  .catch(error => console.error("Error:", error));

fetchFromEntry('entry2', '/orders')
  .then(data => console.log("Entry 2 Data:", data))
  .catch(error => console.error("Error:", error));

```

```jsx
// 커스텀 헤더
axios.defaults.headers.common['Authorization'] = 'Bearer your_token_here';
axios.defaults.headers.post['Content-Type'] = 'application/json';

//쿠키
axios.defaults.withCredentials = true;

//토큰
axios.interceptors.request.use(config => {
  config.headers.Authorization = `Bearer ${localStorage.getItem('token')}`;
  return config;
});

axios.interceptors.response.use(
  response => response,
  async error => {
    if (error.response.status === 401) {
      const newToken = await refreshAuthToken();
      localStorage.setItem('token', newToken);
      error.config.headers['Authorization'] = `Bearer ${newToken}`;
      return axios(error.config);  // 실패했던 요청 재시도
    }
    return Promise.reject(error);
  }
);

//엔트리 2개
import axios, { AxiosInstance } from "axios";

const defaultConfig = {};

const apiRequester: AxiosInstance = axios.create(defaultConfig);
const orderApiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.or/",
  ...defaultConfig,
});
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: "https://cart.baemin.order/",
  ...defaultConfig,
});
```

1. Axios 인터셉터 사용하기
    
    <aside>
    💡
    
    인터셉터 : 모든 요청이나 응답이 실제로 서버에 도달하거나 클라이언트로 돌아오기 전에 특정 작업을 추가할 수 있도록 하는 기능
    
    실행 위치
    
    - 요청 인터셉터 : 서버로 요청 보내기 전 실행 (헤더 등 설정 추가 가능)
    - 응답 인터셉터 : 서버에서 응답이 돌아온 후 실행 (에러 처리, 응답 데이터 가공 가능)
    
    기능
    
    - 한 번 작성 후, 모든 요청 자동 실행 가능 ⇒ 공통 작업 자동화 (특정 헤더 자동 추가)
    </aside>
    
    1. 인터셉터 기능을 `requester`에 따라 비동기 호출 내용을 추가해서 처리
        1. 엔드포인트마다 인증 방식이 다른 경우
        2. 토큰 갱신이 필요한 상황
    
    ```tsx
    import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from "axios";
    
    const getUserToken = () => "";  // 사용자 토큰을 가져오는 함수
    const getAgent = () => "";      // 에이전트 정보를 가져오는 함수
    const getOrderClientToken = () => "";  // Order 클라이언트 토큰을 가져오는 함수
    const orderApiBaseUrl = "";     // Order API 기본 URL
    const orderCartApiBaseUrl = ""; // Order Cart API 기본 URL
    const defaultConfig = {};       // API 요청의 기본 설정 객체
    const httpErrorHandler = () => {};  // HTTP 에러를 처리하는 함수
    
    // 인스턴스 생성
    const apiRequester: AxiosInstance = axios.create({
      baseURL: "https://api.baemin.com",
      timeout: 5000,
    });
    
    //헤더 설정 함수 => 기본 헤더 설정 및 apiRequester와 orderCartApiRequester의 인터셉터로 사용됨
    const setRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
      const config = requestConfig;
      config.headers = {
        ...config.headers,
        "Content-Type": "application/json;charset=utf-8",
        user: getUserToken(),
        agent: getAgent(),
      };
      return config;
    };
    
    // Order API의 요청 시 사용할 헤더 (Order 클라이언트 토큰을 추가) => 엔드 포인트마다 인증 방식이 다른 경우에 해당
    // orderApiRequester의 요청 인터셉터로 설정
    const setOrderRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
      const config = requestConfig;
      config.headers = {
        ...config.headers,
        "Content-Type": "application/json;charset=utf-8",
        "order-client": getOrderClientToken(),
      };
      return config;
    };
    
    // `interceptors` 기능을 사용해 header를 설정하는 기능을 넣거나 에러를 처리할 수 있다
    // setRequestDefaultHeader()가 아니라 setRequestDefaultHeader 함수 참조를 인자로 전달하여 즉시 실행하지 않고, 요청이 발생하때마다 요청 실행 직전에 작업 수행이 가능하도록 한다.
    apiRequester.interceptors.request.use(setRequestDefaultHeader);
    const orderApiRequester: AxiosInstance = axios.create({
      baseURL: orderApiBaseUrl,
      ...defaultConfig,
    });
    
    // 기본 apiRequester와는 다른 header를 설정하는 `interceptors`
    orderApiRequester.interceptors.request.use(setOrderRequestDefaultHeader);
    
    // `interceptors`를 사용해 httpError 같은 API 에러를 처리할 수도 있다
    orderApiRequester.interceptors.response.빌더(
      (response: AxiosResponse) => response,
      httpErrorHandler
    );
    
    const orderCartApiRequester: AxiosInstance = axios.create({
      baseURL: orderCartApiBaseUrl,
      ...defaultConfig,
    });
    
    orderCartApiRequester.interceptors.request.use(setRequestDefaultHeader);
    ```
    
    1. 클래스 형태 구성 ⇒ http 요청을 유연하고 재사용 가능한 방식으로 수행
    
    ```tsx
    import axios, { AxiosPromise } from "axios";
    
    // 임시 타이핑
    export type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
    
    export type HTTPHeaders = any;
    
    export type HTTPParams = unknown;
    
    //
    class API {
      readonly method: HTTPMethod;
    
      readonly url: string;
    
      baseURL?: string;
    
      headers?: HTTPHeaders;
    
      params?: HTTPParams;
    
      data?: unknown;
    
      timeout?: number;
    
      withCredentials?: boolean;
    
      constructor(method: HTTPMethod, url: string) {
        this.method = method;
        this.url = url;
      }
    
      call<T>(): AxiosPromise<T> {
        const http = axios.create();
        // 만약 `withCredential`이 설정된 API라면 아래 같이 인터셉터를 추가하고, 아니라면 인터셉터 를 사용하지 않음
        if (this.withCredentials) {
          http.interceptors.response.use(
            (response) => response,
            (error) => {
              if (error.response && error.response.status === 401) {
                /* 에러 처리 진행 */
              }
              return Promise.reject(error);
            }
          );
        }
        return http.request({ ...this });
      }
    }
    
    export default API;
    
    // 사용 예시
    const orderApi = new API("GET", "/orders", "https://api.order.com");
    const cartApi = new API("POST", "/cart", "https://api.cart.com");
    
    // 각 API 호출
    orderApi.call().then((response) => console.log("Order API:", response));
    cartApi.call().then((response) => console.log("Cart API:", response));
    
    ```
    
    1. 빌더 패턴 이용
        1. 장점 : 요청 옵션에 따라 인터셉터를 다르게 만들어야 할 때, 엔트리별로 설정 함수와 인스턴스를 만드는 대신, **메서드 체이닝**으로 쉽게 인스턴스 생성 가능
        
        ```tsx
        import API, { HTTPHeaders, HTTPMethod, HTTPParams } from "./7.1.4-2";
        
        const apiHost = "";
        
        class APIBuilder {
          private _instance: API;
        
          constructor(method: HTTPMethod, url: string, data?: unknown) {
            this._instance = new API(method, url);
            this._instance.baseURL = apiHost;
            this._instance.data = data;
            this._instance.headers = {
              "Content-Type": "application/json; charset=utf-8",
            };
            this._instance.timeout = 5000;
            this._instance.withCredentials = false;
          }
        
          static get = (url: string) => new APIBuilder("GET", url);
        
          static put = (url: string, data: unknown) => new APIBuilder("PUT", url, data);
        
          static post = (url: string, data: unknown) =>
            new APIBuilder("POST", url, data);
        
          static delete = (url: string) => new APIBuilder("DELETE", url);
        
          baseURL(value: string): APIBuilder {
            this._instance.baseURL = value;
            return this;
          }
        
          headers(value: HTTPHeaders): APIBuilder {
            this._instance.headers = value;
            return this;
          }
        
          timeout(value: number): APIBuilder {
            this._instance.timeout = value;
            return this;
          }
        
          params(value: HTTPParams): APIBuilder {
            this._instance.params = value;
            return this;
          }
        
          data(value: unknown): APIBuilder {
            this._instance.data = value;
            return this;
          }
        
          withCredentials(value: boolean): APIBuilder {
            this._instance.withCredentials = value;
            return this;
          }
        
          build(): API {
            return this._instance;
          }
        }
        
        export default APIBuilder;
        
        //사용 예시
        import APIBuilder from "./7.1.4-3";
        
        // ex
        type Response<T> = { data: T };
        type JobNameListResponse = string[];
        
        const fetchJobNameList = async (name?: string, size?: number) => {
          const api = APIBuilder.get("/apis/web/jobs")
            .withCredentials(true) // 이제 401 에러가 나는 경우, 자동으로 에러를 탐지하는 인터셉터를 사용하게 된다
            .params({ name, size }) // body가 없는 axios 객체도 빌더 패턴으로 쉽게 만들 수 있다 => 필요한 속성만 선별하여 설정 가능(유연성)
            .build();
          const { data } = await api.call<Response<JobNameListResponse>>();
          return data;
        };
        ```
        
        b. 단점 : 보일러 플레이트 코드(반복적으로 사용되는 기본적인 코드)가 많다. ⇒ 옵션 객체를 통한 일괄설정
        
    
    ```tsx
    interface APIOptions {
      method?: string;
      url?: string;
      headers?: Record<string, string>;
      params?: Record<string, any>;
      data?: unknown;
      timeout?: number;
    }
    
    class APIBuilder {
      private options: APIOptions = {};
    
      setOptions(options: APIOptions): this {
        this.options = { ...this.options, ...options };
        return this;
      }
    
      build(): API {
        return new API(this.options);
      }
      
    }
    ```
    
2. API 응답 요청하기

서버에서 오는 응답 통일 시, 주의점

- 응답이 없을 수 있는 API 처리 까다로움
    - body없이 상태코드만 반환이 되는 UPDATE, CREATE 요청의 경우 타입을 강제하려고 하면 오류가 발생하거나 처리로직이 더 복잡해질 수 있음 ⇒ 응답이 없는 경우 기본값 설정, 응답 유무에 따라 타입 분리 등

⇒ 예측할 수 없는 응답 형식이나 상황에 유연하게 대처하기 위해 Response 타입은 apiRequester가 모르게 관리되는 것이 좋다.

```tsx
interface response {
  data: {
    cartItems: CartItem[];
    forPass: unknown;
  };
}

type ForPass = {
  type: "A" | "B" | "C";
};

const isTargetValue = () => (data.forPass as ForPass).type === "A";
```

⇒ forPass는 시스템 내에서 안정적으로 보장된 타입이 아니기 때문에 해당 값에 의존하지 않고 사용을 피해야하는데 써야하는 값이라면 타입 가드를 활용해서 타입 안전성을 확보 후 사용해야한다.

1. 뷰 모델 사용하기

서버 스펙이 자주 바뀜에 따라 변하는 API 응답은 뷰모델을 사용하여 API 변경에 따른 범위 한정해야한다.

- 장점 : 백엔드/UI 추가 로직 설정 없이 도메인 개념 추가하여 프론트엔드에서 간편하게 사용 가능, 응답이 변경되어도 UI 깨지지 않게 개발 가능

```tsx
interface ListResponse {
  items: ListItem[];
}

const fetchList = async (filter?: ListFetchFilter): Promise<ListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis/get-list-summaries")
    .call<Response<ListResponse>>();

  return { data };
};

const ListPage: React.FC = () => {
  const [totalItemCount, setTotalItemCount] = useState(0);
  const [items, setItems] = useState<ListItem[]>([]);

  useEffect(() => {
    // 예시를 위한 API 호출과 then 구문
    fetchList(filter).then(({ items }) => {
      setTotalItemCount(items.length);
      setItems(items);
    });
  }, []);

  return (
    <div>
      <Chip label={totalItemCount} />
      <Table items={items} />
    </div>
  );
};

```

```tsx
// 기존 ListResponse에 더 자세한 의미를 담기 위한 변화
interface JobListItemResponse {
  name: string;
}

interface JobListResponse {
  jobItems: JobListItemResponse[];
}

class JobList {
// API 응답에 없는 도메인 개념 추가 시, 백엔드, UI 로직 추가 없이 추가 가능
  readonly totalItemCount: number;
  readonly items: JobListItemResponse[];
  
  constructor({ jobItems }: JobListResponse) {
    // jobItems 배열 길이를 기반으로 totalItemCount를 계산하여 추가
    this.totalItemCount = jobItems.length;
    this.items = jobItems;
  
	  // API 응답이 바뀌어도 UI 깨지지 않게 개발 가능
	  // jobItems가 누락된 경우에도 빈 배열로 초기화하여 UI가 깨지지 않도록 함
    //this.items = response.jobItems ?? [];
    //this.totalItemCount = this.items.length;
  }
}

const fetchJobList = async (
  filter?: ListFetchFilter
): Promise<JobListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis/get-list-summaries")
    .call<Response<JobListResponse>>();

//응답 데이터에 대한 구조적 의미 부여
  return new JobList(data);
};

const ListPage: React.FC = () => {
  const [totalItemCount, setTotalItemCount] = useState(0);
  const [items, setItems] = useState<JobListItemResponse[]>([]);

  useEffect(() => {
    fetchJobList(filter).then((jobList) => {
      setTotalItemCount(jobList.totalItemCount);
      setItems(jobList.items);
    });
  }, []);

  return (
    <div>
      <Chip label={totalItemCount} />
      <Table items={items} />
    </div>
  );
};

```

- 단점 : 추상황 레이어 추가로 인해 타입이 많아지며 코드가 복잡해질 수 있다. API 응답에 없는 새로운 필드를 만들어서 사용했을 때, 의사소통 문제 가능성
- 절충안
    - 필요한 곳에만 뷰모델 부분적으로 사용
    - 충분한 의사소통을 통해 API 응답 변화 최소화
    - 필드 추가 대신 getter등의 함수를 추가
1. Superstruct를 사용해 런타임에서 응답 타입 검증

<aside>
💡

타입스크립트는 컴파일 타임 오류를 감지할 수 있기 때문에 실행 시점의 데이터 구조를 보장할 수 없습니다. 타입스크립트는 런타임에 타입 오류를 감지하지 못하므로 데이터가 없거나 예상한 형태가 아닌 경우 자바스크립트 엔진에서 런타임 오류가 발생할 수 있습니다.

그렇기 때문에 코드가 실제로 실행될 때 오류 ( 서버에서 받아온 응답 데이터, 외부 소스에서 제공되는 데이터 검사)를 위해서는 런타임 응답 타입을 검증하는 라이브러리를 활용하여 **예상한 필드가 누락되거나 잘못된 타입의 값이 포함된 경우** 런타입에서 이를 감지하고 오류를 반환해 예기지 못한 동작을 방지할 수 있습니다.

</aside>

![image.png](https://github.com/user-attachments/assets/ca6eb830-a33b-4356-bcbc-fe0f97554239)

## API 상태 관리하기

<aside>
💡

실제 API 요청 코드를 컴포넌트 내부에서 비동기 함수로 직접 호출하지 않는 이유

- 코드 유지 보수성, 응집성 높임
- 테스트, 오류 처리 간편화
</aside>

1. 상태 관리 라이브러리에서 호출
    
    Redux : 1. 스토어 생성 → 2. 액션 디스패치 → 3. 리듀서로 상태 업데이트(새로운 상태 객체 반환) → 4. 구독 및 상태 변경 감지 → 5. 미들웨어로 부가 작업 처리
    https://github.com/user-attachments/assets/ca6eb830-a33b-4356-bcbc-fe0f97554239
    ![image.png]()
    
    ```tsx
    import { useEffect } from "react";
    import { useDispatch, useSelector } from "react-redux";
     // store 변경과 api 요청을 동시에 하고 있음.
    export function useMonitoringHistory() {
      const dispatch = useDispatch();
      // 전역 Store 상태(RootState)에서 필요한 데이터만 가져온다
      const searchState = useSelector(
        (state: RootState) => state.monitoringHistory.searchState
      );
      // history 내역을 검색하는 함수, 검색 조건이 바뀌면 상태를 갱신하고 API를 호출한다
      // api 호출과 redux 상태 관리를 하나의 함수에서 처리 => 테스트와 유지보수 어려움
      const getHistoryList = async (
        newState: Partial<MonitoringHistorySearchState>
      ) => {
        const newSearchState = { ...searchState, ...newState };
        dispatch(monitoringHistorySlice.actions.changeSearchState(newSearchState));
        const response = await getHistories(newSearchState); // 비동기 API 호출하기 dispatch(monitoringHistorySlice.actions.fetchData(response));
      };
    
      return { searchState, getHistoryList };
    }
    ```
    
    ```tsx
    enum ApiCallStatus {
      Request,
      None,
    }
    
    const API = axios.create();
    // 요청 시작과 완료 시점에 스토어 상태 변경
    const setAxiosInterceptor = (store: EnhancedStore) => {
      API.interceptors.request.use(
        (config: AxiosRequestConfig) => {
          const { params, url, method } = config;
          store.dispatch(
            // API 상태 저장을 위해 redux reducer setApiCall 함수를 사용한다 // 상태가 요청됨인 경우 API가 Loading 중인 상태
            setApiCall({
              status: ApiCallStatus.Request, // API 호출 상태를 요청됨으로 변경
              urlInfo: { url, method },
            })
          );
          return config;
        },
        (error) => Promise.reject(error)
      );
      // onSuccess 시 인터셉터로 처리한다
      API.interceptors.response.use(
        (response: AxiosResponse) => {
          const { method, url } = response.config;
          store.dispatch(
            setApiCall({
              status: ApiCallStatus.None, // API 호출 상태를 요청되지 않음으로 변경
              urlInfo: { url, method },
            })
          );
          return response?.data?.data || response?.data;
        },
        (error: AxiosError) => {
          const {
            config: { url, method },
          } = error;
          store.dispatch(
            setApiCall({
              status: ApiCallStatus.None, // API 호출 상태를 요청되지 않음으로 변경
              urlInfo: { url, method },
            })
          );
          return Promise.reject(error);
        }
      );
    };
    ```
    
    - 단점 : 비동기 처리 함수를 호출하기 위해 액션이 추가될 때마다 스토어와 상태가 계속 늘어나며 전역 상태 관리자가 모든 비동기 상태에 접근하고 변경이 가능해지며 쓸데없는 비동기 통신 및 의도치 않은 상태 변경이 발생할 수 있다.
2. 훅으로 호출
    
    <aside>
    💡
    
    > 캐시를 이용하여 비동기 함수를 호출 ⇒ 상태 관리 라이브러리에서 발생했던 의도치 않은 상태 변경을 방지하는데 도움이 된다.
    > 
    - 훅을 사용하면 각 컴포넌트가 독립적인 상태과 비동기 로직을 가져서 캐시된 데이터를 통해 필요한 경우만 상태를 업데이트할 수 있다.
    - 캐시된 데이터를 훅을 통해 관리되면 특정 데이터가 이미 존재할 경우 api 호출 X
    - 생명주기 관리 훅과 함께 사용되며 특정 조건에서만 비동기 함수 실행하며 불필요한 상태 업데이트 방지
    </aside>
    
    - 장점 : 상태 관리 라이브러리의 단점(스토어 비대해짐, 전역 상태 복잡해짐 등) 보완
    - React-Query
        - 서버에서 가져온 비동기 데이터의 캐싱과 상태 관리를 최적화하는 라이브러리
        - 데이터를 사용하는 동안에만 데이터에 대한 참조를 유지하고 사용하지 않으면 캐시에서 제거
        - 비동기 데이터와 관련된 상태에 특화 (로딩, 에러, 성공, 갱신 상태 등)

## API 에러 헨들링

1. 타입 가드 활용하기
    
    ```tsx
    interface ErrorResponse {
      status: string;
      serverDateTime: string;
      errorCode: string;
      errorMessage: string;
    }
    
    function isServerError(error: unknown): error is AxiosError<ErrorResponse> {
      return axios.isAxiosError(error);
    }
    
    const onClickDeleteHistoryButton = async (id: string) => {
      try {
        await axios.post("https://....", { id });
    
        alert("주문 내역이 삭제되었습니다.");
      } catch (error: unknown) {
        if (isServerError(e) && e.response && e.response.data.errorMessage) {
          // 서버 에러일 때의 처리임을 명시적으로 알 수 있다 setErrorMessage(e.response.data.errorMessage);
          return;
        }
        setErrorMessage("일시적인 에러가 발생했습니다. 잠시 후 다시 시도해주세요");
      }
    };
    ```
    
2. 에러 서브클래싱(기존 클래스를 확장하여 새로운 클래스를 만드는 과정)하기

```tsx
class OrderHttpError extends Error {
  private readonly privateResponse: AxiosResponse<ErrorResponse> | undefined;

  constructor(message?: string, response?: AxiosResponse<ErrorResponse>) {
    super(message);
    this.name = "OrderHttpError";
    this.privateResponse = response;
  }

  get response(): AxiosResponse<ErrorResponse> | undefined {
    return this.privateResponse;
  }
}

class NetworkError extends Error {
  constructor(message = "") {
    super(message);
    this.name = "NetworkError";
  }
}

class UnauthorizedError extends Error {
  constructor(message: string, response?: AxiosResponse<ErrorResponse>) {
    super(message, response);
    this.name = "UnauthorizedError";
  }
}
```

1. 인터셉터를 활용한 에러처리

```tsx
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  (error) => {
    // 401 에러인 경우 로그인 페이지로 이동
    if (error.response && error.response.status === 401) {
      window.location.href = `${backOfficeAuthHost}/login?targetUrl=${window.location.href}`;
    }
    return Promise.reject(error);
  };
};

orderApiRequester.interceptors.response.use(
  (response: AxiosResponse) => response,
  httpErrorHandler
);
```

1. 에러 바운더리를 활용한 에러처리
    
    리액트 컴포넌트 트리 하위에 있는 컴포넌트에서 발생한 에러를 캐치하고 해당 에러를 가장 가까운 부모 에러 바운더리에서 처리 (대신 에러 처리 또는 공통 처리) 
    
    [에러가 바운더리가 캐치를 못하는 경우](https://www.youtube.com/watch?v=v69zRgDCjjs) (비동기 에러, SSR 에러, React 외부 에러, 타입 에러 등)
    
2. 상태 관리 라이브러리에서 에러처리 ⇒ store 에 에러를 상태로 처리 후 컴포넌트에서 활용
3. react-query 를 활용한 에러 처리
4. 커스텀 에러 (Axios 등 라이브러리 기능 활용 가능)

## API 모킹

<aside>
💡

모킹 : 서버가 완성되지 않았을 때, FE 테스트를 위해 가짜 모듈을 활용하여 개발 가능

</aside>

1. json 파일 불러오기
2. NextApiHandler 활용하기
3. API 요청 핸들러에 분기 추가하기 
4. axios-mock-adapter로 모킹하기 (요청 가로채서 응답값 대신 반환)
5. 목업 사용 여부 제어하기 (로컬과 dev/운영환경 구분)

![image.png](https://github.com/user-attachments/assets/e131cf68-66b2-4875-880a-4ce994e369e5)
