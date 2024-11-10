# 9장 훅

2024.11.10 dasom



## 9.1 리액트 훅

클래스 컴포넌트에서는 componentDidMount, componentDidUpdate 등 하나의 생명주기 함수에서 상태 업데이트에 따른 로직을 관리한다. 하지만 프로젝트 규모가 커지면서 개발 생산성이 떨어지는 일이 발생했고, 훅이 도입되어 함수형 컴포넌트에서 쉽게 구현이 가능해졌다.

### 1. useState

함수 컴포넌트에서 상태를 관리하기 위해 활용하는 훅이다.

타입스크립트와 함께 사용해, 예상치 못한 상태 변경으로 인해 발생할 수 있는 사이드 이펙트를 방지할 수 있다.

```ts
function useState<S>(
initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];

type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState:S) => S);
```



### 2. 의존성 배열을 사용하는 훅

**useEffect와 useLayoutEffect**

* useEffect

  : 렌더링 이후 리액트 함수 컴포넌트에 어떤 일을 수행해야하는지 알려주기 위해 활용하는 훅이다.

  useEffect 의존성에 객체나 배열을 넣을 때는 얕은 비교로만 판단되기 때문에 사용에 유의해야 한다.

> useEffect와 비동기 함수
>
> useEffect의 콜백함수는 Promise 타입은 반환하지 않는다. useEffect에서 비동기 함수를 호출할 수 있다면 경쟁 상태를 불러일으킬 수 있기 때문이다.
>
> 경쟁 상태는, 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생 가능한 문제이다. 이러한 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램 동작이 원하지 않는 방향으로 흘러갈 수 있다.



* useLayoutEffect

  : useEffect와 유사한 훅으로, useEffect는 렌더링이 모두 완료된 후에 실행된다면 useLayoutEffect는 화면에 해당 컴포넌트가 그려지기 전에 콜백함수를 실행해 첫 번째 렌더링 때 빈 값이 뜨는 경우 등을 방지할 수 있다.



**useMemo와 useCallback**

두 훅 모두 이전에 생성된 값 혹은 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 해주는 훅이다. 어떤 값을 계산하는 데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용하게 사용할 수 있다.

의존성에 대해 얕은 비교를 수행하며, 과도하게 메모이제이션 하지 않도록 사용해야한 다는 점을 유의해야 한다.



**useRef**

DOM을 직접 선택해야하는 경우 사용하는 훅이다.

useRef는 넣어주는 인자 타입에 따라 반환되는 타입이 달라진다.

```ts
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
};

interface RefObject<T> {
  readonly current: T | null;
}
```

MutableRefObject의 current는 변경할 수 있는 값이 되어 ref.current의 값이 바뀌는 사이드 이펙트가 발생할 수 있다.

RefObject의 current는 readonly로 값을 변경할 수 없다. 



**자식 컴포넌트에 ref 전달하기**

ref를 일반적인 props로 넘겨주는 방식으로 전달하면 에러 메세지가 발생하게 된다. ref라는 속성의 이름은 DOM 요소 접근이라는 특수한 목적으로 사용되기 때문이다. ref를 prop으로 전달하려면 forwardRef를 사용해야 한다.

```tsx
interface Props {
  name: string;
}

const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
  return (
  <div>
  	<label>{props.name}</label>
    <input ref={ref} />
  </div>
  )
})
```

유의사항: ForwardedFef에는 오직 MutableRefObject만 들어올 수 있다. MutableRefObject가 RefObject보다 넓은 범위의 타입을 가지기 때문에, 부모 컴포넌트에서 ref를 어떻게 선언했는지와 관계 없이 자식 컴포넌트가 해당 ref를 수용할 수 있다.



**useInperativeHandle**

ForwardRefRenderFunction과 함께 쓸 수 있는 훅으로, 이 훅을 활용해 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 메서드를 호출할 수 있게 된다.

예시-1. 자식 컴포넌트)

```tsx
type CreateFormHandle = Pick<HTMLFormElement, "submit">;

type CreateFormProps = {
  defaultValues?: CreateFormValue;
};

const JobCreateForm: React.ForwardRefRenderFunction<CreateFormHandle, CreateFormProps> = (props, ref) => {
  // submit 함수 커스터마이징
  useImperativeHandle(ref, () => {
    submit: () => {
      /* submit 작업 진행 */
    }
  })
}
```

예시-2. 부모 컴포넌트)

```tsx
const CreatePage: React.Fc = () => {
  const refForm = useRef<CreateFormHandle>(null);
  
  const handleSubmitButtonClick = () => {
    // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근 가능
    refForm.current?.submti();
  }
}
```



**useRef의 여러가지 특성**

* useRef로 관리되는 변수는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않는다.
* 리액트 컴포넌트의 상태는 상태 변경 함수를 호출하고 렌더링된 이후에 업데이트된 상태를 조회할 수. ㅣㅆ다. 반면 useRef로 관리되는 변수는 값을 설정한 후 즉시 조회할 수 있다.

> 🤖 Input 공통 컴포넌트를 구현할 때, useState vs useRef?
>
>  `input`의 `value`처럼 변경 사항이 즉각적으로 UI에 반영되어야 하는 경우라면 **`useState`가 적합**합니다.



> 훅의 규칙
>
> * 훅은 항상 최상위 레벨에서 호출되어야 한다. 
>   * 조건문, 반복문, 중첩 함수, 클래스 등 내부에서는 훅을 호출하지 않아야 한다.
> * 훅은 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.



## 커스텀 훅

리액트 컴포넌트 내에서만 사용가능하며, 이름은 반드시 use로 시작해야 한다.

### 1. 나만의 훅 만들기

예시- useInput)

```tsx
import { useState } from "react";

const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);
  
  const onChane = (e) => {
    setValue(e.target.value);
  }
  
  return {value, onChange};
}
```

예시 - useInput 사용)

```tsx
const MyComponent = () => {
  const {value, onChange} = useInput("");
  
  return (
  <div>
    <h1>{value}</h1>
  	<input onChange={onChange} value={text} />  
  </div>
  )
}
```



### 2. 타입스크립트로 커스텀 훅 강화하기

예시- useInput)

```tsx
import { useState, useCallback, ChangeEvent } from "react";

const useInput = (initialValue: string) => {
  const [value, setValue] = useState(initialValue);
  
  const onChane = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  }, []);
  
  return {value, onChange};
}
```

