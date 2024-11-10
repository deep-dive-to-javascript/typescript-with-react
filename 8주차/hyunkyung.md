## 9장 훅
### 9.1 리액트 훅
- 리액트에 훅이 추가되기 전에는 클래스 컴포넌트에서만 상태를 가질 수 있었음
- 클래스 컴포넌트에서는 componentDidMount, componentDidUpdate와 같이 하나의 생명주기 함수에서만 상태 업데이트에 따른 로직을 실행할 수 있었음
- 프로젝트 규모가 커지면서 상태를 스토어에 연결하거나 비슷한 로직을 가진 상태 업데이트 및 사이드 이펙트 처리가 불편해짐
- 또한 모든 상태를 하나의 함수 내에서 처리하다보니 관심사가 뒤섞이게 되었고 상태에 따른 테스트나 잘못 발생한 사이드 이펙트의 디버깅이 어려워짐

```js
componentDidMount() {
  this.props.updateCurrentPage(routeName);
  this.didFocusSubscription = this.props.navigation.addListener('focus', () => {/*
add focus handler to navigation */});
  this.didBlurSubscription = this.props.navigation.addListener('blur', () => {/* add
blur handler to navigation */});
}

componentWillUnmount() {
  if (this.didFocusSubscription != null) {
    this.didFocusSubscription();
  }
  if (this.didBlurSubscription != null) {
    this.didBlurSubscription();
  }
  if (this._screenCloseTimer != null) {
    clearTimeout(this._screenCloseTimer);
    this._screenCloseTimer = null;
  }
}

componentDidUpdate(prevProps) {
  if (this.props.currentPage != routeName) return;

  if (this.props.errorResponse != prevProps.errorResponse) {/* handle error response
*/}
  else if (this.props.logoutResponse != prevProps.logoutResponse) {/* handle logout
response */}
  else if (this.props.navigateByType != prevProps.navigateByType) {/* handle
navigateByType change */}

  // Handle other prop changes here
}
```
- componentWillUnmount에서는 componentDidMount에서 정의한 컴포넌트가 DOM에서 해제될 때 실행되어야 할 여러 사이드 이펙트 함수를 호출
- 만약 componentWillUnmount에서 실행되어야 할 사이드 이펙트가 하나 빠졌다면 componentDidMount와 비교해가며 어떤 함수가 빠졌는지 찾아야 할 것
- 또한 props 변경에 대한 디버깅을 수행하려면 componentDidUpdate에서 내가 원하는 props 변경 조건문이 나올 때까지 코드를 찾아봐야 함
- 리액트 훅이 도입되면서 함수 컴포넌트에서도 클래스 컴포넌트와 같이 컴포넌트의 생명주기에 맞춰 로직을 실행할 수 있게 되었음
- 이에 따라 비즈니스 로직을 재사용하거나 작은 단위로 코드를 분할하여 테스트하는 게 용이해졌으며 사이드 이펙트와 상태를 관심사에 맞게 분리하여 구성할 수 있게 됨

#### 1. useState
- 리액트 함수 컴포넌트에서 상태를 관리하기 위해 useState 훅을 활용할 수 있음
- useState의 타입 정의는 다음과 같음
```ts
function useState<S>(
  initialState: S | (() => S)
  ): [S, Dispatch<SetStateAction<S>>];
  
type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);
```
- useState가 반환하는 튜플의 첫번째 요소는 제네릭으로 지정한 S 타입, 두번째 요소는 상태를 업데이트할 수 있는 Dispatch 타입의 함수
- Dispatch 함수의 제네릭으로 지정한 SetStateAction에는 useState로 관리할 상태 타입인 S 또는 이전 상태 값을 받아 새로운 상태를 반환하는 함수인 (prevState: S) => S가 들어갈 수 있음

```tsx
import { useState } from "react";

const MemberList = () => {
  const [memberList, setMemberList] = useState([
    {
      name: "KingBaedal",
      age: 10,
      },
    {
      name: "MayBaedal",
      age: 9,
    },
  ]);
  
  // 🚨 addMember 함수를 호출하면 sumAge는 NaN이 된다
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);
  
  const addMember = () => {
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11,
      },
    ]);
  };
};
```
- 기존 memberList 배열 요소에는 없는 agee라는 잘못된 속성이 포함된 객체가 추가됨으로서 sumAge 변수가 NaN이 되는 예상치 못한 사이드 이펙트가 발생한다.
- 타입스크립트를 사용하면 이런 에러 방지 가능
```tsx
- import { useState } from "react";

interface Member {
  name: string;
  age: number;
}

const MemberList = () => {
  const [memberList, setMemberList] = useState<Member[]>([]);
  
  // member의 타입이 Member 타입으로 보장된다
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);
  
  const addMember = () => {
  // 🚨 Error: Type ‘Member | { name: string; agee: number; }’
  // is not assignable to type ‘Member’
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11,
      },
    ]);
  };

  return (
    // ...
  );
};
```

#### 2. 의존성 배열을 사용하는 훅
**useEffect와 useLayoutEffect**

- 렌더링 이후 리액트 함수 컴포넌트에 어떤 일을 수행해야 하는지 알려주기 위해 useEffect 훅 활용

```ts
function useEffect(effect: EffectCallback, deps?: DependencyList): void;
type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

- useEffect의 첫번째 인자이자 effect의 타입인 EffectCallback은 Destructor를 반환하거나 아무것도 반환하지 않는 함수
- Promise 타입은 반환하지 않으므로 useEffect의 콜백 함수에는 비동기 함수가 들어갈 수 없음
  - useEffect에서 비동기 함수를 호출할 수 있다면 경쟁 상태를 불러일으킬 수 있기 때문
  - 경쟁 상태란 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제
  - 이 경우 첫번째로 useEffect가 실행되었을 때 fetch의 응답이 오고 있는 중에 dependency가 변경되어 두번째 useEffect가 실행되는 경우가 있을 수 있습니다. 
    - 두번째 useEffect에서 응답이 먼저 도착했을 때 먼저 state를 업데이트하게 되고 이후 첫번째 useEffect에서 받은 응답을 덮어쓰게 됩니다. 

```js
useEffect(() => {
  const fetchData = async () => {
    const response = await fetch("https://example.com/api/data");
    const data = await response.json();
    setState(data); // 상태 업데이트
  };
  
  fetchData();
}, [dependency]);
```

- useEffect의 두번째 인자인 deps는 옵셔널하게 제공되며 effect가 수행되기 위한 조건을 나열
- 이때 useEffect는 deps가 변경되면 실행이 되고 이를 얕은 비교로 판단하기 때문에 객체나 배열의 경우 참조값이 변경되면 콜백 함수가 실행
    - 따라서 원치않는 렌더링이 발생할 수 있음
- 따라서 실제 사용하는 값을 useEffect와 deps에서 사용해야 함
- 이러한 특징은 useMemo나 useCallback과 같은 다른 훅에서도 동일하게 적용됨

```ts
type SomeObject = {
  name: string;
  id: string;
};

interface LabelProps {
  value: SomeObject;
}

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    // value.name과 value.id를 사용해서 작업한다
  }, [value]);
  
  // ...
};
```

- useEffect는 Destructor를 반환하는데 이것은 컴포넌트가 마운트 해제될 때 실행하는 함수
    - deps가 빈 배열이라면 useEffect의 콜백 함수는 컴포넌트가 처음 렌더링될때만 실행되며, 이때의 Destructor는 컴포넌트가 마운트 해제될 때 실행
    - 그러나 deps 배열이 존재한다면 배열의 값이 변경될 때마다 Desturctor가 실행됨
      - 이전 작업을 취소하거나 정리하는 용도로 사용
- useEffect와 비슷한 역할을 하는 훅으로 useLayoutEffect가 있음
```ts
type DependencyList = ReadonlyArray<any>;

function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;
```
- useEffect는 앞서 살펴본 componentDidUpdate와 같은 기존 생명주기 함수와는 다르게, 레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행됨

```jsx
const [name, setName] = useState("");

useEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래의 setName()을 실행한다고 생각하자
  setName("배달이");
}, []);

return (
  <div>
    {`안녕하세요, ${name}님!`}
  </div>
);
```

- 이와 같은 코드를 실행하면 처음에는 "안녕하세요, 님!"이 렌더링되고, setName이 실행되어 "안녕하세요, 배달이님!"이 렌더링됨
- 만약 name을 지정하는 setName이 오랜 시간이 걸린 후에 실행된다면 사용자는 빈 이름을 오랫동안 보게 됨
- useLayoutEffect를 사용해서 화면에 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행하여 사용자에게 빈 화면을 보여주지 않음

**useMemo와 useCallback**
- 두 훅 모두 이전에 생성된 값 또는 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 해주는 훅
- 어떤 값을 계산하는 데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 useMemo나 useCallback을 유용하게 사용할 수 있음
```ts
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;
```

- 앞에서 살펴본 useEffect와 비슷한 주의사항이 존재한다.
- useMemo와 useCallback을 사용해서 과도하게 메모이제이션하면 컴포넌트의 성능 향상이 보장되지 않을 수 있음

#### 3. useRef
- 리액트 애플리케이션에서 dom 요소에 직접 접근해야 하는 경우에 사용

```tsx
import { useRef } from "react";

const MyComponent = () => {
  const ref = useRef<HTMLInputElement>(null);
  
  const onClick = () => {
    ref.current?.focus();
  };
  
  return (
    <>
      <button onClick= {onClick}>ref에 포커스!</button>
      <input ref= {ref} />
    </>
  );
};

export default MyComponent;
```

- useRef는 세 종류의 타입 정의를 가지고 있음.

```ts
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}

interface RefObject<T> {
  readonly current: T | null;
}
```

- useRef는 RefObject 또는 MutableRefObject를 반환
- MutableRefObject의 current는 값을 변경할 수 있음 
  - 만약 HTMLInputElement | null을 넣어주었다면 해당 useRef는 첫번째 타입을 따를 것이고 ref.current의 값이 바뀌는 사이드 이펙트가 발생할 수 있음
- 반면 RefObject는 current의 값이 읽기 전용이기 때문에 값을 변경할 수 없음

**자식 컴포넌트에 ref 전달하기**
- 기본 HTML 요소가 아닌, 리액트 컴포넌트에 ref를 전달할 수도 있다.
- 이때 ref를 일반적인 props로 넘겨주는 방식으로 전달하면 브라우저에서 경고 메세지를 띄운다
- ref 속성의 이름은 리액트에서 DOM 요소 접근이라는 특수한 목적으로 사용되기 때문에 props를 넘겨주는 방식으로 전달할 수 없음
- 리액트 컴포넌트에서 ref를 prop으로 전달하기 위해서는 forwardref를 사용해야 함
- forwardRef의 타입 정의는 다음과 같음
```ts
function forwardRef<T, P = {}>(
  render: ForwardRefRenderFunction<T, P>
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;

interface ForwardRefRenderFunction<T, P = {}> {
  (props: P, ref: ForwardedRef<T>): ReactElement | null;
  displayName?: string | undefined;
  defaultProps?: never | undefined;
  propTypes?: never | undefined;
}

type ForwardedRef<T> =
        | ((instance: T | null) => void)
        | MutableRefObject<T | null>
        | null;
```

- 여기서 주목할 것은 MutableRefObject를 사용했다는 것

**useImperativeHandle**
- useImperativeHandle은 ForwardRefRenderFunction과 함께 쓸 수 있는 훅
- 이 훅을 사용하면 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징된 메서드를 호출할 수 있게 됨

```tsx
// <form> 태그의 submit 함수만 따로 뽑아와서 정의한다
type CreateFormHandle = Pick<HTMLFormElement, "submit">;

type CreateFormProps = {
  defaultValues?: CreateFormValue;
};

const JobCreateForm: React.ForwardRefRenderFunction<CreateFormHandle,
  CreateFormProps> = (props, ref) => {
  // useImperativeHandle Hook을 사용해서 submit 함수를 커스터마이징한다
  useImperativeHandle(ref, () => ({
    submit: () => {
      /* submit 작업을 진행 */
    }
  }));
  
  // ...
}

const CreatePage: React.FC = () => {
  // `CreateFormHandle` 형태를 가진 자식의 ref를 불러온다
  const refForm = useRef<CreateFormHandle>(null);

  const handleSubmitButtonClick = () => {
    // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근할 수 있다
    refForm.current?.submit();
  };

  // ...
};
```

**useRef의 여러 가지 특성**
- useRef로 관리되는 변수는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않는다. 이런 특성을 활용하면 상태가 변경되더라도 불필요한 리렌더링을 피할 수 있다.
- 리액트 컴포넌트의 상태는 상태 변경함수를 호출하고 렌더링된 이후에 업데이트된 상태를 조회할 수 있다. 반면 useRef로 관리되는 변수는 값을 설정한 후 즉시 조회할 수 있다.

```tsx
type BannerProps = {
  autoplay: boolean;
};

const Banner: React.FC<BannerProps> = ({ autoplay }) => {
  const isAutoPlayPause = useRef(false);
  
  if (autoplay) {
    // keepAutoPlay 같이 isAutoPlay가 변하자마자 사용해야 할 때 쓸 수 있다
    const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current;

    // ...
  }
  return (
    <>
      {autoplay && (
        <button
          aria-label="자동 재생 일시 정지"
          // isAutoPlayPause는 사실 렌더링에는 영향을 미치지 않고 로직에만 영향을 주므로 상태로 사용해서 불필요한 렌더링을 유발할 필요가 없다
          onClick={() => { isAutoPlayPause.current = true }}
        />
      )}
    </>
  );
};

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    // value.name과 value.id를 사용해서 작업한다
  }, [value]);
  
  // ...
};
```

### 9.2 커스텀 훅
#### 1. 나만의 훅 만들기
- 커스텀 훅은 use라는 접두사로 시작하는 함수
```jsx
import { useState } from "react";

const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);
  
  const onChange = (e) => {
    setValue(e.target.value);
  };

  return { value, onChange };
};

const MyComponent = () => {
  const { value, onChange } = useInput("");

  return (
          <div>
            <h1>{value}</h1>
            <input onChange= {onChange} value= {text} />
          </div>
  );
};

export default App;
```

#### 2. 타입스크립트로 커스텀 훅 강화하기
```ts
import { useState, useCallback } from "react";

// 🚨 Parameter ‘initialValue’ implicitly has an ‘any’ type.ts(7006)
const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);

  // 🚨 Parameter ‘e’ implicitly has an ‘any’ type.ts(7006)
  const onChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);

  return { value, onChange };
};

export default useInput;
```
- 기존 코드를 .ts 파일에 작성하면 에러가 발생

```ts
import { useState, useCallback, ChangeEvent } from "react";

// ✅ initialValue에 string 타입을 정의
const useInput = (initialValue: string) => {
  const [value, setValue] = useState(initialValue);

  // ✅ 이벤트 객체인 e에 ChangeEvent<HTMLInputElement> 타입을 정의
  const onChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  }, []);

  return { value, onChange };
};

export default useInput;
```

- e의 타입을 알기는 쉽지 않음. IDE를 활용하면 tsc가 현재 사용하고 있는 이벤트 객체의 타입을 유추해서 알려주므로 유용