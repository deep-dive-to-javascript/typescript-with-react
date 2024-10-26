## 7장 비동기 호출
### 7.1 API 요청
1. fetch로 API 요청하기
- 아래와 같은 코드로 fetch를 사용하여 API 요청하려고 할 때 fetch 함수에 들어가는 값들이 변경되거나 추가됐을 때 비동기 호출 코드를 수정해야하는 번거로움 발생 
```tsx
import React, { useEffect, useState } from "react";

const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0);

  useEffect(() => {
    fetch("https://api.baemin.com/cart")
      .then((response) => response.json())
      .then(({ cartItem }) => {
        setCartCount(cartItem.length);
      });
  }, []);

  return <>{/*  cartCount 상태를 이용하여 컴포넌트 렌더링 */}</>;
};
```

2. 서비스 레이어로 분리하기
- 여러 API 요청 정책이 추가되어 코드가 변경될 수 있다는 것을 감안한다면, 비동기 호출 코드는 컴포넌트 영역에서 분리되어 다른 영역(서비스 레이어)에서 처리되어야 함
- 앞 예시를 기반으로 설명하면 fetch 함수를 호출하는 부분이 서비스 레이어로 이동하고, 컴포넌트는 서비스 레이어의 비동기 함수를 호출하여 그 결과를 받아 렌더링하는 흐름
- 하지만 단순히 fetch 함수를 분리하는 것만으로는 API 요청 정책이 추가되는 것을 해결하기 어려움
  - fetch 함수에서 타임아웃을 설정하기 위해서 아래와 같이 구현해야 하는데 다양한 API 정책이 추가될 수 있는데 모두 구현하는 것은 번거로운 일

```ts
async function fetchCart() {
  const controller = new AbortController();

  const timeoutId = setTimeout(() => controller.abort(), 5000);

  const response = await fetch("https://api.baemin.com/cart", {
    signal: controller.signal,
  });

  clearTimeout(timeoutId);

  return response;
}
```

3. Axios 활용하기
- fetch는 내장 라이브러리이기 때문에 따로 설치할 필요가 없지만 추가 기능은 직접 구현해야 하는 번거로움이 있음
- 이러한 번거로움 때문에 Axios 라이브러리를 사용하여 API 요청을 보다 쉽게 처리할 수 있음

```ts
import axios, { AxiosInstance, AxiosPromise } from "axios";

export type FetchCartResponse = unknown;
export type PostCartRequest = unknown;
export type PostCartResponse = unknown;

export const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000,
});

export const fetchCart = (): AxiosPromise<FetchCartResponse> =>
  apiRequester.get<FetchCartResponse>("cart");

export const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<PostCartResponse> =>
  apiRequester.post<PostCartResponse>("cart", postCartRequest);
```

- API entry가 2개 이상일 경우에는 인스턴스 따로 구성
```ts
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

4. Axios 인터셉터 사용하기
- 인터셉터 기능을 사용하여 requester에 따라 호출 내용을 추가해서 처리할 수 있음
- 또한 API 에러를 처리할 때 하나의 에러 객체로 묶어서 처리할 수도 있음

```ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from "axios";

const getUserToken = () => "";
const getAgent = () => "";
const getOrderClientToken = () => "";
const orderApiBaseUrl = "";
const orderCartApiBaseUrl = "";
const defaultConfig = {};
const httpErrorHandler = () => {};

const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000,
});

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
apiRequester.interceptors.request.use(setRequestDefaultHeader);
const orderApiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});
// 기본 apiRequester와는 다른 header를 설정하는 `interceptors`
orderApiRequester.interceptors.request.use(setOrderRequestDefaultHeader);
// `interceptors`를 사용해 httpError 같은 API 에러를 처리할 수도 있다
orderApiRequester.interceptors.response.use(
  (response: AxiosResponse) => response,
  httpErrorHandler
);
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: orderCartApiBaseUrl,
  ...defaultConfig,
});
orderCartApiRequester.interceptors.request.use(setRequestDefaultHeader);
```

- 이와 달리 요청 옵션에 따라 다른 인터셉터를 만들기 위해 빌더 패턴을 추가하여 APIBuilder 같은 클래스 형태로 구성하기도 함
```ts
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
```

- 기본 API 클래스로 실제 호출 부분을 구성하고 위와 같은 API를 호출하기 위한 래퍼를 빌더 패턴으로 만든다
```ts
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
```

- 이와 같은 패턴으로 제공한 APIBuilder를 사용하는 코드는 다음과 같다
```ts
import APIBuilder from "./7.1.4-3";

// ex
type Response<T> = { data: T };
type JobNameListResponse = string[];

const fetchJobNameList = async (name?: string, size?: number) => {
  const api = APIBuilder.get("/apis/web/jobs")
    .withCredentials(true) // 이제 401 에러가 나는 경우, 자동으로 에러를 탐지하는 인터셉터를 사용하게 된다
    .params({ name, size }) // body가 없는 axios 객체도 빌더 패턴으로 쉽게 만들 수 있다
    .build();
  const { data } = await api.call<Response<JobNameListResponse>>();
  return data;
};
```

5. API 응답 타입 지정하기
- 같은 서버에서 오는 응답의 형태는 대체로 통일되어 있어서 앞서 소개한 API의 응답 값은 하나의 Response 타입으로 묶일 수 있음
```ts
import { AxiosPromise } from "axios";
import {
  FetchCartResponse,
  PostCartRequest,
  PostCartResponse,
  apiRequester,
} from "./7.1.3-1";

export interface Response<T> {
  data: T;
  status: string;
  serverDateTime: string;
  errorCode?: string; // FAIL, ERROR errorMessage?: string; // FAIL, ERROR
}
const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> =>
  apiRequester.get<Response<FetchCartResponse>>("cart");

const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<Response<PostCartResponse>> =>
  apiRequester.post<Response<PostCartResponse>>("cart", postCartRequest);
```

- 이때 주의할 점은 Response 타입을 apiRequester 내에서 처리하게 되면 UPDATE나 CREATE 같이 응답이 없을 수 있는 API를 처리하기 까다로워진다는 것
```ts
import { AxiosPromise } from "axios";
import { FetchCartResponse, apiRequester } from "./7.1.3-1";
import { Response } from "./7.1.5-1";

const updateCart = (
  updateCartRequest: unknown
): AxiosPromise<Response<FetchCartResponse>> => apiRequester.get("cart");
```

- 따라서 Response 타입은 apiRequester가 모르게 관리되어야 함
- API 요청 및 응답 값 중에서는 하나의 API 서버에서 다른 API 서버로 넘겨주기만 하는 값도 존재할 수 있음
- 해당 값이 로직에 영향을 주지 않는 경우에는 Unknown 타입을 사용하여 알 수 없는 값임을 표현
- 필요한 경우엔 프론트 로직에서 써야하는 값에 대해서만 타입을 선언 후 사용

```ts
interface response {
  data: {
    cartItems: CartItem[];
    forPass: unknown;
  };
}
```

6. 뷰 모델 사용하기
- 서버 스펙 변경에 대비하여 뷰모델을 사용하여 API 변경에 따른 범위를 한정해주는 것이 좋음
- 특정 객체 리스트를 조회하여 렌더링해야하는 화면의 api는 아래와 같이 구성될 것
- 만약 api 응답의 items 인자를 더 정확한 개념으로 나타내기 위해 jobItems나 cartItems 같은 이름으로 수정하면 해당 컴포넌트도 수정해야 함
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

- 이러한 문제를 해결하기 위한 방법으로 뷰 모델 도입

```tsx
// 기존 ListResponse에 더 자세한 의미를 담기 위한 변화
interface JobListItemResponse {
  name: string;
}

interface JobListResponse {
  jobItems: JobListItemResponse[];
}

class JobList {
  readonly totalItemCount: number;
  readonly items: JobListItemResponse[];
  constructor({ jobItems }: JobListResponse) {
    this.totalItemCount = jobItems.length;
    this.items = jobItems;
  }
}

const fetchJobList = async (
  filter?: ListFetchFilter
): Promise<JobListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis/get-list-summaries")
    .call<Response<JobListResponse>>();

  return new JobList(data);
};
```

- 그러나 뷰 모델 방식에서도 문제점은 있음
  - API를 추가할떄마다 뷰모델이 늘어남
  - API 응답에는 없는 새로운 필드를 만들어 사용할때 서버와 클라이언트간의 의사소통이 어려워질 수 있다는 것
- 이러한 문제점을 해결하기 위해 꼭 필요한 곳에만 뷰 모델을 부분적으로 사용하기, 백엔드와 클라이언트 개발자가 충분히 소통하기, 뷰 모델에 필드를 추가하는 대신에 Getter 함수를 추가하기 등의 방법이 있다
- 개발 단계에서는 API 응답 형식이 자주 바뀜 -> 런타임에 API 응답 타입 오류를 방지하려면 superstruct 같은 라이브러리 사용

7. Superstruct를 사용해 런타임에서 응답 타입 검증하기
- Superstruct의 간단한 예시 
```ts
import {
  assert,
  is,
  validate,
  object,
  number,
  string,
  array,
} from "superstruct";

const Article = object({
  id: number(),
  title: string(),
  tags: array(string()),
  author: object({
    id: number(),
  }),
});

const data = {
  id: 34,
  title: "Hello World",
  tags: ["news", "features"],
  author: {
    id: 1,
  },
};

assert(data, Article);
is(data, Article);
validate(data, Article);
```

- assert, is, validate의 공통점은 데이터 정보를 담은 data 변수와 데이터 명세를 가진 스키마인 Article을 인자로 받아 데이터가 스키마와 부합하는지 검사한다는 것
- 차이점
  - assert는 유효하지 않을 경우 에러를 던짐
  - is는 유효성 검사 결과에 따라 true나 false를 반환
  - validate은 [error, data] 형식의 튜플을 반환. 유효하지 않을 때는 에러 값이 반환, 유효한 경우에는 첫번째 요소로 undefined, 두번째 요소로 data value가 반환

- 타입스크립트에서 어떻게 사용할 수 있는지 알아보자
- 아래와 같이 Infer를 사용하여 기존 타입 선언 방식과 동일하게 타입 선언
```ts
import { Infer, number, object, string } from "superstruct";

const User = object({
  id: number(),
  email: string(),
  name: string(),
});

type User = Infer<typeof User>;
```

- 이렇게 하면 User 타입은 아래와 같이 정의됨
```ts
type User = {
  id: number;
  email: string;
  name: string;
};

import { assert } from "superstruct";

// user가 User 타입인지 검사
function isUser(user: User) {
  assert(user, User);
  console.log("적절한 유저입니다.");
}

const user_A = {
  id: 4,
  email: "test@woowahan.email",
  name: "woowa",
};

isUser(user_A); // 적절한 유저입니다.

const user_B = {
  id: 5,
  email: "wrong@woowahan.email",
  name: 4,
};

isUser(user_B); // error TS2345: Argument of type '{ id: number; email: string; name: number; }' is not assignable to parameter of type '{ id: number; email: string; name: string; }'
```

8. 실제 API 응답 시의 Superstruct 활용 사례
```ts
import { assert } from "superstruct";

interface ListItem {
  id: string;
  content: string;
}

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

function isListItem(listItems: ListItem[]) {
  listItems.forEach((listItem) => assert(listItem, ListItem));
}
```

### 7.2 API 상태 관리하기
- 실제 API를 요청하는 코드는 컴포넌트 내에서 비동기 함수를 직접 호출하지는 않음
- 비동기 API를 호출하기 위해서는 API의 성공&실패에 따른 상태가 관리되어야하므로, 상태 관리 라이브러리의 액션이나 훅과 같이 재정의된 형태를 사용해야 함

1. 상태 관리 라이브러리에서 호출하기
- 상태 관리 라이브러리의 비동기 함수들은 서비스 코드를 사용해서 비동기 상태를 변화시킬 수 있는 함수 제공
- 컴포넌트는 이러한 함수를 사용하여 상태를 구독하며, 상태가 변경될 때 컴포넌트를 다시 렌더링하는 방식으로 동작
- Redux 예시

```ts
import { useEffect } from "react";
import { useDispatch, useSelector } from "react-redux";

export function useMonitoringHistory() {
  const dispatch = useDispatch();
  // 전역 Store 상태(RootState)에서 필요한 데이터만 가져온다
  const searchState = useSelector(
    (state: RootState) => state.monitoringHistory.searchState
  );
  // history 내역을 검색하는 함수, 검색 조건이 바뀌면 상태를 갱신하고 API를 호출한다
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

- getHistoryList 함수에서는 dispatch 코드를 제외하더라도 다음과 같이 API 호출과 상태 관리 코드를 작성해야 함
```ts
enum ApiCallStatus {
  Request,
  None,
}

const API = axios.create();

const setAxiosInterceptor = (store: EnhancedStore) => {
  API.interceptors.request.use(
    (config: AxiosRequestConfig) => {
      const { params, url, method } = config;
      store.dispatch(
        // API 상태 저장을 위해 redux reducer `setApiCall` 함수를 사용한다 // 상태가 `요청됨`인 경우 API가 Loading 중인 상태
        setApiCall({
          status: ApiCallStatus.Request, // API 호출 상태를 `요청됨`으로 변경
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
          status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
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
          status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
          urlInfo: { url, method },
        })
      );
      return Promise.reject(error);
    }
  );
};
```

- Redux는 비동기 상태가 아닌 전역 상태를 위해 만들어진 라이브러리이기 때문에 미들웨어라고 불리는 여러 도구를 도입하여 비동기 상태를 관리한다.
- 따라서 보일러플레이트 코드가 많아지는 등 간편하게 비동기 상태를 관리하기 어려운 상황도 발생한다.
- MobX 같은 라이브러리에서는 이러한 불편함을 개선하기 위해 비동기 콜백 함수를 분리하여 액션으로 만들거나 runInAction과 같은 메서드를 사용하여 상태변경을 처리함

```ts
import { runInAction, makeAutoObservable } from "mobx";
import type Job from "models/Job";

class JobStore {
  job: Job[] = [];
  constructor() {
    makeAutoObservable(this);
  }
}

type LoadingState = "PENDING" | "DONE" | "ERROR";

class Store {
  job: Job[] = [];
  state: LoadingState = "PENDING";
  errorMsg = "";

  constructor() {
    makeAutoObservable(this);
  }

  async fetchJobList() {
    this.job = [];
    this.state = "PENDING";
    this.errorMsg = "";
    try {
      const projects = await fetchJobList();
      runInAction(() => {
        this.projects = projects;
        this.state = "DONE";
      });
    } catch (e) {
      runInAction(() => {
        this.state = "ERROR";
        this.errorMsg = e.message;
      });
    }
  }
}
```

- 모든 상태 관리 라이브러리에서 비동기 처리 함수를 호출하기 위해 액션이 추가될 때마다 관련된 스토어나 상태가 계속 늘어남
- 이로 인한 가장 큰 문제점은 전역 상태 관리자가 모든 비동기 상태에 접근하고 변경할 수 있다는 것
- 의도치 않은 상태 변경이 발생할 수 있으며, 이로 인해 디버깅이 어려워질 수 있음

2. 훅으로 호출하기
- react-query나 useSwr 같은 훅을 사용한 방법은 상태 변경 라이브러리를 사용한 방식보다 훨씬 간단함
- 이러한 훅은 캐시를 사용하여 비동기 함수를 호출하며, 상태 관리 라이브러리에서 발생했던 의도치 않은 상태 변경을 방지하는 데 도움이 됨

```ts
// Job 목록을 불러오는 훅
const useFetchJobList = () => {
  return useQuery(["fetchJobList"], async () => {
    const response = await JobService.fetchJobList(); // View Model을 사용해서 결과
    return new JobList(response);
  });
};

// Job 1개를 업데이트하는 훅
const useUpdateJob = (
  id: number,
  // Job 1개 update 이후 Query Option
  { onSuccess, ...options }: UseMutationOptions<void, Error, JobUpdateFormValue>
): UseMutationResult<void, Error, JobUpdateFormValue> => {
  const queryClient = useQueryClient();

  return useMutation(
    ["updateJob", id],
    async (jobUpdateForm: JobUpdateFormValue) => {
      await JobService.updateJob(id, jobUpdateForm);
    },
    {
      onSuccess: (
        data: void, // updateJob의 return 값은 없다 (status 200으로만 성공 판별) values: JobUpdateFormValue,
        context: unknown
      ) => {
        // 성공 시 ‘fetchJobList’를 유효하지 않음으로 설정 queryClient.invalidateQueries(["fetchJobList"]);
        onSuccess && onSuccess(data, values, context);
      },
      ...options,
    }
  );
};
```

- 이후 컴포넌트에서는 일반적인 훅을 호출하는 것처럼 사용하면 됨
- JobList 컴포넌트가 반드시 최신 상태를 표현하려면 폴링이나 웹소켓 등의 방법을 사용해야 함

```tsx
const JobList: React.FC = () => {
  // 비동기 데이터를 필요한 컴포넌트에서 자체 상태로 저장
  const {
    isLoading,
    isError,
    error,
    refetch,
    data: jobList,
  } = useFetchJobList();

  // 간단한 Polling 로직, 실시간으로 화면이 갱신돼야 하는 요구가 없어서 // 30초 간격으로 갱신한다
  useInterval(() => refetch(), 30000);

  // Loading인 경우에도 화면에 표시해준다
  if (isLoading) return <LoadingSpinner />;

  // Error에 관한 내용은 11.3 API 에러 핸들링에서 더 자세하게 다룬다
  if (isError) return <ErrorAlert error={error} />;

  return (
    <>
      {jobList.map((job) => (
        <Job job={job} />
      ))}
    </>
  );
};
```

### 7.3 API 에러 핸들링
1. 타입가드 활용하기
```ts
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

2. 에러 서브클래싱하기
- 실제 요청을 처리할 때 단순한 서버 에러도 발생하지만 인증 정보 에러, 네트워크 에러, 타임아웃 에러 같은 다양한 에러가 발생하기도 함
- 이를 더욱 명시적으로 표시하기 위해 서브클래싱을 활용

```ts
const getOrderHistory = async (page: number): Promise<History> => {
  try {
    const { data } = await axios.get(`https://some.site?page=${page}`);
    const history = await JSON.parse(data);

    return history;
  } catch (error) {
    // 어떤 에러가 발생한 것인지 판단할 수가 없음
    alert(error.message);
  }
};
```

- 이러한 문제를 해결하기 위해 에러 서브클래싱을 사용하여 에러를 더욱 명시적으로 표현할 수 있음

```ts
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

- 아래와 같이 에러 객체를 상속한 클래스를 정의한다

```ts
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  let promiseError: Promise<Error>;

  if (axios.isAxiosError(error)) {
    if (Object.is(error.code, "ECONNABORTED")) {
      promiseError = Promise.reject(new TimeoutError());
    } else if (Object.is(error.message, "Network Error")) {
      promiseError = Promise.reject(new NetworkError());
    } else {
      const { response } = error as AxiosError<ErrorResponse>;
      switch (response?.status) {
        case HttpStatusCode.UNAUTHORIZED:
          promiseError = Promise.reject(
            new UnauthorizedError(response?.data.message, response)
          );
          break;
        default:
          promiseError = Promise.reject(
            new OrderHttpError(response?.data.message, response)
          );
      }
    }
  } else {
    promiseError = Promise.reject(error);
  }

  return promiseError;
};
```

- 아래와 같이 활용할 수 있음

```ts
const alert = (meesage: string, { onClose }: { onClose?: () => void }) => {};

const onActionError = (
  error: unknown,
  params?: Omit<AlertPopup, "type" | "message">
) => {
  if (error instanceof UnauthorizedError) {
    onUnauthorizedError(
      error.message,
      errorCallback?.onUnauthorizedErrorCallback
    );
  } else if (error instanceof NetworkError) {
    alert("네트워크 연결이 원활하지 않습니다. 잠시 후 다시 시도해주세요.", {
      onClose: errorCallback?.onNetworkErrorCallback,
    });
  } else if (error instanceof OrderHttpError) {
    alert(error.message, params);
  } else if (error instanceof Error) {
    alert(error.message, params);
  } else {
    alert(defaultHttpErrorMessage, params);
  }

  const getOrderHistory = async (page: number): Promise<History> => {
    try {
      const { data } = await fetchOrderHistory({ page });
      const history = await JSON.parse(data);

      return history;
    } catch (error) {
      onActionError(error);
    }
  };
};
```

3. 인터셉터를 활용한 에러 처리
```ts
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

4. 에러 바운더리를 활용한 에러처리
- 에러 바운더리는 리액트 컴포넌트 트리에서 에러가 발생할 때 공통으로 에러를 처리하는 리액트 컴포넌트
- 에러 바운더리를 사용하면 리액트 컴포넌트 트리 하위에 있는 컴포넌트에서 발생한 에러를 캐치하고, 해당 에러를 가장 가까운 부모 에러 바운더리에서 처리할 수 있게 한다.
```tsx
import React, { ErrorInfo } from "react";
import ErrorPage from "pages/ErrorPage";

interface ErrorBoundaryProps {}

interface ErrorBoundaryState {
  hasError: boolean;
}

class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(): ErrorBoundaryState {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    this.setState({ hasError: true });

    console.error(error, errorInfo);
  }

  render(): React.ReactNode {
    const { children } = this.props;
    const { hasError } = this.state;

    return hasError ? <ErrorPage /> : children;
  }
}

const App = () => {
  return (
    <ErrorBoundary>
      <OrderHistoryPage />
    </ErrorBoundary>
  );
};
```

5. 상태 관리 라이브러리에서의 에러 처리
- 앞서 잠깐 살펴본 Redux의 에러처리방법은 아래와 같다
```ts
// API 호출에 관한 api call reducer
const apiCallSlice = createSlice({
  name: "apiCall",
  initialState,
  reducers: {
    setApiCall: (state, { payload: { status, urlInfo } }) => {
      /* API State를 채우는 logic */
    },
    setApiCallError: (state, { payload }: PayloadAction<any>) => {
      state.error = payload;
    },
  },
});

const API = axios.create();

const setAxiosInterceptor = (store: EnhancedStore) => {
  /* 중복 코드 생략 */
  // onSuccess시 처리를 인터셉터로 처리한다
  API.interceptors.response.use(
    (response: AxiosResponse) => {
      const { method, url } = response.config;

      store.dispatch(
        setApiCall({
          status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
          urlInfo: { url, method },
        })
      );

      return response?.data?.data || response?.data;
    },
    (error: AxiosError) => {
      // 401 unauthorized
      if (error.response?.status === 401) {
        window.location.href = error.response.headers.location;

        return;
      }
      // 403 forbidden
      else if (error.response?.status === 403) {
        window.location.href = error.response.headers.location;
        return;
      }
      // 그 외에는 화면에 alert 띄우기
      else {
        message.error(`[서버 요청 에러]: ${error?.response?.data?.message}`);
      }

      const {
        config: { url, method },
      } = error;

      store.dispatch(
        setApiCall({
          status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
          urlInfo: { url, method },
        })
      );

      return Promise.reject(error);
    }
  );
};
```

- 이렇게 저장된 에러는 컴포넌트에서 사용할 수 있음
```ts
const fetchMenu = createAsyncThunk(
  FETCH_MENU_REQUEST,
  async ({ shopId, menuId }: FetchMenu) => {
    try {
      const data = await api.fetchMenu(shopId, menuId);
      return data;
    } catch (error) {
      setApiCallError({ error });
    }
  }
);
```

- MobX를 사용하고 있다면 주로 스토어에서 에러 핸들링

```tsx
class JobStore {
  jobs: Job[] = [];
  state: LoadingState = "PENDING"; // "PENDING" | "DONE" | "ERROR"; errorMsg = "";

  constructor() {
    makeAutoObservable(this);
  }

  async fetchJobList() {
    this.jobs = [];
    this.state = "PENDING";
    this.errorMsg = "";

    try {
      const projects = await fetchJobList();

      runInAction(() => {
        this.projects = projects;
        this.state = "DONE";
      });
    } catch (e) {
      runInAction(() => {
        // 에러 핸들링 코드를 작성
        this.state = "ERROR";
        this.errorMsg = e.message;
        showAlert();
      });
    }
  }

  get isLoading(): boolean {
    return state === "PENDING";
  }
}

const JobList = (): JSX.Element => {
  const [jobStore] = useState(() => new JobStore());

  if (jobStore.job.isLoading) {
    return <Loader />;
  }

  return (
    <>
      {jobStore.jobs.map((job) => (
        <Item job={job} />
      ))}
    </>
  );
};
```

6. react-query를 활용한 에러 처리
```tsx
const JobComponent: React.FC = () => {
  const { isError, error, isLoading, data } = useFetchJobList();
  if (isError) {
    return (
      <div>{`${error.message}가 발생했습니다. 나중에 다시 시도해주세요.`}</div>
    );
  }
  if (isLoading) {
    return <div>로딩 중입니다.</div>;
  }
  return (
    <>
      {data.map((job) => (
        <JobItem key={job.id} job={job} />
      ))}
    </>
  );
};
```

7. 그 밖의 에러처리
- 비즈니스 로직에서의 유효성 검증에 의해 추가된 커스텀 에러는 200 응답과 함께 응답 바디에 별도의 상태 코드를 전달하기도 함
- 예를 들어 장바구니에서 주문을 생성하는 API가 다음과 같은 커스텀 에러를 반환한다고 해보자

```text
{
  "httpStatus": 200,
  {
    "status": "C20005",
    "message": "장바구니에 품절된 메뉴가 있습니다."
  }
}
```

- 이 에러를 처리하기 위해 요청 함수 내에서 조건문으로 status를 비교할 수 있음

```ts
const successHandler = (response: CreateOrderResponse) => {
  if (response.status === "SUCCESS") {
    // 성공 시 진행할 로직을 추가한다
    return;
  }
  throw new CustomError(response.status, response.message);
};
const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post("https://...", data);

    successHandler(response);
  } catch (error) {
    errorHandler(error);
  }
};
```
- 만약 커스텀 에러를 사용하고 있는 요청을 일괄적으로 에러로 처리하고 싶다면 Axios 등의 라이브러리 기능을 활용하면 됨

```ts
export const apiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});

export const httpSuccessHandler = (response: AxiosResponse) => {
  if (response.data.status !== "SUCCESS") {
    throw new CustomError(response?.data.message, response);
  }

  return response;
};

apiRequester.interceptors.response.use(httpSuccessHandler, httpErrorHandler);

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post("https://...", data);

    successHandler(response);
  } catch (error) {
    // status가 SUCCESS가 아닌 경우 에러로 전달된다
    errorHandler(error);
  }
};
```

### 7.4 API 모킹
- API 모킹은 프론트엔드 개발자가 백엔드 개발자의 API를 기다리지 않고 프론트엔드 개발을 진행할 수 있게 해주는 방법
- 이외에도 이슈가 생겼을 때 charles 등의 도구를 활용하면 응답 값을 그대로 복사하여 이슈 발생 상황을 재현하는 데 도움이 됨
- 우형 프론트에서는 axios-mock-adapter, NextApi-Handler 등을 활용하여 API를 모킹해서 사용

1. JSON 파일 불러오기
- 간단한 조회만 필요한 경우에는 *.Json 파일을 만들거나 자바스크립트 파일 안에 JSON 형식의 정보를 저장하고 익스포트 해주는 방식을 사용

2. NextAPIHandler 활용
- NextJS를 사용하고 있다면 NextAPIHandler 활용할 수 있음

```ts
// api/mock/brand
import { NextApiHandler } from "next";

const BRANDS: Brand[] = [
  {
    id: 1,
    label: "배민스토어",
  },
  {
    id: 2,
    label: "비마트",
  },
];

const handler: NextApiHandler = (req, res) => {
  // request 유효성 검증
  res.json(BRANDS);
};

export default handler;
```

3. API 요청 핸들러에 분기 추가하기
- 요청 경로를 수정하지 않고 필요한 경우에만 실제 요청을 보내고 그 외에는 목업을 사용하여 개발하고 싶다면 아래와 같이 처리 가능

```ts
const mockFetchBrands = (): Promise<FetchBrandsResponse> =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        status: "SUCCESS",
        message: null,
        data: [
          {
            id: 1,
            label: "배민스토어",
          },
          {
            id: 2,
            label: "비마트",
          },
        ],
      });
    }, 500);
  });

const fetchBrands = () => {
  if (useMock) {
    return mockFetchBrands();
  }

  return requester.get("/brands");
};
```

4. axios-mock-adapter로 모킹하기
- axios-mock-adapter는 axios를 확장하여 axios 요청을 가로채서 응답을 조작할 수 있게 해주는 라이브러리

```ts
// mock/index.ts
import axios from "axios";
import MockAdapter from "axios-mock-adapter";
import fetchOrderListSuccessResponse from "fetchOrderListSuccessResponse.json";

interface MockResult {
  status?: number;
  delay?: number;
  use?: boolean;
}

const mock = new MockAdapter(axios, { onNoMatch: "passthrough" });

export const fetchOrderListMock = () =>
  mock.onGet(/\/order\/list/).reply(200, fetchOrderListSuccessResponse);

// fetchOrderListSuccessResponse.json
{
    "data": [
        {
            "orderNo": "ORDER1234", "orderDate": "2022-02-02", "shop": {
            "shopNo": "SHOP1234",
            "name": "가게이름1234" },
            "deliveryStatus": "DELIVERY"
        },
    ]
}
```

- 아래처럼 응답 처리하는 부분을 별도 함수로 구현하면 여러 Mock 함수에서 사용할 수 있음

```ts
export const lazyData = (
  status: number = Math.floor(Math.random() * 10) > 0 ? 200 : 200,
  successData: unknown = defaultSuccessData,
  failData: unknown = defaultFailData,
  time = Math.floor(Math.random() * 1000)
): Promise<any> =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve([status, status === 200 ? successData : failData]);
    }, time);
  });

export const fetchOrderListMock = ({
  status = 200,
  time = 100,
  use = true,
}: MockResult) =>
  use &&
  mock
    .onGet(/\/order\/list/)
    .reply(() =>
      lazyData(status, fetchOrderListSuccessResponse, undefined, time)
    );
```

5. 목업 사용 여부 제어하기
- 로컬에서는 목업을 사용하고 dev나 운영 환경에서 사용하지 않으려면 아래와 같이 설정해주면 됨

```ts
const useMock = Object.is(REACT_APP_MOCK, "true");

const mockFn = ({ status = 200, time = 100, use = true }: MockResult) =>
  use &&
  mock.onGet(/\/order\/list/).reply(
    () =>
      new Promise((resolve) =>
        setTimeout(() => {
          resolve([
            status,
            status === 200 ? fetchOrderListSuccessResponse : undefined,
          ]);
        }, time)
      )
  );

if (useMock) {
  mockFn({ status: 200, time: 100, use: true });
}
```

- 스크립트 실행시 구분짓고자 한다면 아래처럼 하면 됨
```json
// package.json

{
  "scripts": {
    "start:mock": "REACT_APP_MOCK=true npm run start",
    "start": "REACT_APP_MOCK=false npm run start"
  }
}
```

- MSW를 사용하는 방식도 있다. 서비스워커를 활용하므로 개발환경과 운영환경을 분리할 수 있다.

### 활용
- axios 인스턴스와 api 정의
- error handler
- mock data