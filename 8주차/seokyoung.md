## 1. 리액트 훅

- 함수 컴포넌트에서도 클래스 컴포넌트와 같이 컴포넌트의 생명주기에 맞춰 로직 실행 가능
- 비즈니스 로직 재사용 가능
- 작은 단위로 코드 분할하여 테스트 용이
- 사이드 이펙트와 상태를 관심사에 맞게 분리하여 구성 가능

### `useState`

```tsx
funtcion useState<S>(initialState: S | (() => S)): [S, Dispatch<SetState<S>>]

type Dispatch<A> = (value: A) => void
type SetStateAction<S> = S | ((prevState: S) => S)
```

- `[S, Dispatch<SetState<S>>]`
  - `S`: 관리할 상태의 타입
  - `Dispatch<SetState<S>>`: `SetStateAction`에서 상태를 그대로 받거나 이전 상태 값을 받아 새로운 상태 값으로 반환하는 업데이터 함수를 받는 `Dispatch` 타입의 함수

#### 타입스크립트를 `useState`에 적용하기

```tsx
import { useState } from 'react'

interface Member {
  name: string
  age: number
}

const MemberList = () => {
  const [memberList, setMemberList] = useState<Member[]>([
    {
      name: 'KingBaedal',
      age: 10,
    },
    {
      name: 'MayBaedal',
      age: 9,
    },
  ])

  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0)

  const addMember = () => {
    setMemberList([
      ...memberList,
      {
        name: 'DokgoBaedal',
        agee: 11, // Error: Type 'Member | { name: string; agee: number; } is not assignable to type 'Member'
      },
    ])
  }
}

export default MemberList
```

- 배열 `memberList`의 객체 요소를 `Member` 타입으로 정의하여 `setMemberList`로 추가하는 새 객체의 타입 보장
- `setMemberList` 호출 부분에서 추가하려는 새 객체의 타입을 확인하여 컴파일타임에 타입 에러를 발견할 수 있음

> [!NOTE]
>
> #### 의존성 배열을 사용하는 훅
>
> - `useEffect` / `useLayoutEffet`
> - `useMemo` / `useCallback`

### `useEffect`

```tsx
function useEffect(effect: EffectCallback, deps?: DependencyList): void
type DependencyList = ReadonlyArray<any>
type EffectCallback = () => void | Destructor
```

- `EffectCallback`: `effect` 인자의 타입으로 `Destructor` 타입을 반환하거나 아무것도 반환하지 않는 함수 타입
- `DependencyList`: 의존성 배열 인자인 `deps`의 타입으로, 읽기 전용 배열
  - 옵셔널하게 제공되며, `effect`가 수행되기 위한 조건이 나열된 배열
  - 얕은 비교를 수행하기 때문에 객체나 배열을 넣는 경우 실제로 사용하는 값을 의존성 배열에 넣어줘야 함

#### `usEffect`의 반환 타입

- `useEffect`는 `Promise` 타입을 반환하지 않으므로 `useEffect`의 콜백 함수에는 비동기 함수가 들어갈 수 없음
  - `useEffect`에서 비동기 함수 호출이 가능하면 경쟁 상태를 불러일으킬 수 있기 때문

> [!NOTE]
>
> #### 경쟁 상태
>
> - 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제
> - 실행 순서나 타이밍을 예측할 수 없게 되어 원하지 않는 프로그램 동작이 발생할 수 있음

- `Destructor`: 컴포넌트 마운트 해제될 때 실행하는 함수
  - `deps`가 빈 배열일 경우: 컴포넌트가 처음 렌더링될 때만 `useEffect` 콜백 함수 실행하고 `Destructor`(클린업 함수)는 컴포넌트가 마운트 해제될 때 실행
  - `deps` 배열이 존재할 경우: 배열의 값이 변경될 때마다 `Destructor` 실행

> [!NOTE]
>
> #### `Destructor`
>
> ```tsx
> declare const UNDEFINED_VOID_ONLY: unique symbol // 고유한 타입으로 다른 값과 혼동될 가능성이 없는 상수
> /**
>  * The function returned from an effect passed to useEffect,
>  * which can be used to clean up the effect when the component unmounts.
>  */
> type Destructor = () => void | { [UNDEFINED_VOID_ONLY]: never }
> /**
>  * (1) void를 반환하는 함수
>  *	(2) { [UNDEFINED_VOID_ONLY]: never } 형태의 객체
>  *	- never 타입을 이용해 불가능한 상태를 나타냄
>  *	- 이 객체가 실제로 반환되는 상황은 없도록 설정한 것
>  *  - never는 도달할 수 없는 상태를 의미하기 때문에 → 타입 체크에서 특정 조건을 강제할 때 사용
>  */
> ```

### `useLayoutEffect`

```tsx
type DependencyList = ReadonlyArray<any>

function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void
```

- 화면에 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행하기 때문에 첫 번째 렌더링 때 빈 화면이 뜨는 것을 방지
  - 레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행되는 `useEffect` 특성 보완

### `useMemo`

- **값**을 메모이제이션하여 컴포넌트가 다시 렌더링될 때 매번 계산하지 않도록 하는데 사용
- `deps` 배열을 가지고 있으므로 해당 의존성이 변경되면 값을 다시 계산

```tsx
// `useMemo` will only recompute the memoized value when one of the `deps` has changed.
function useMemo<T>(factory: () => T, deps: DependencyList): T
```

### `useCallback`

- **함수**를 메모이제이션하여 함수가 변경되지 않도록 하는데 사용
- `<T extends Function>`: 함수를 저장하기 위해 제네릭의 기본 타입을 지정
- `deps` 배열을 가지고 있으므로 해당 의존성이 변경되면 값을 다시 계산

```tsx
// `useCallback` will return a memoized version of the callback that only changes if one of the `inputs` has changed.
function useCallback<T extends Function>(callback: T, deps: DependencyList): T
```

> 불필요한 곳에 사용하지 않도록 주의 → 과도한 메모이제이션은 컴포넌트의 성능 향상 보장할 수 없음

### `useRef`

- 포커스 설정, 스크롤 등 DOM을 직접 선택해야 하는 경우 사용

```tsx
import { useRef } from 'react'

const MyComponent = () => {
  const ref = useRef<HTMLInputElement>(null)

  const onClick = () => {
    ref.current.focus()
  }

  return (
    <>
      <button onClick={onClic}>ref에 포커스</button>
      <input ref={ref} />
    </>
  )
}
```

#### `useRef`의 타입 정의

- `useRef`는 `MutableRefObject` 또는 `RefObject`를 반환

```tsx
function useRef<T>(initialValue: T): MutableRefObject<T>
function useRef<T>(initialValue: T | null): RefObject<T>
function useRef<T = undefined>(): MutableRefObject<T | undefined>
```

- `MutableRefObject`의 `current`는 값을 변경 가능
- 만약 `null`을 허용하기 위해 `useRef`의 제네릭에 `HTMLInputElement | null` 타입을 넣어주었다면, 해당 `useRef`는 첫 번째 타입 정의를 따름
- 이때 `MutableRefObject`의 `current`는 변경할 수 있는 값이 되어 `ref.current`의 값이 바뀌는 사이드 이펙트가 발생할 수 없음

```tsx
interface MutableRefObject<T> {
  current: T
}
```

- 반면 `RefObject`의 `current`는 readonly로 값을 변경할 수 없음
- `useRef`의 제네릭에 `HTMLInputElement`을 넣고, 인자에 `null`을 넣어주었다면 두 번째 타입 정의를 따름
- `RefObject`를 반환하여 `ref.current` 값을 임의로 변경할 수 없게 됨

```tsx
interface RefObject<T> {
  /**
   * The current value of the ref.
   */
  readonly current: T | null
}
```

#### 자식 컴포넌트에 ref 전달하기

- 리액트 컴포넌트에 ref를 전달해서 사용 가능
- ref를 자식 컴포넌트의 prop으로 전달해서 prop로 넘겨주는 방식으로 전달하는 게 아닌 `forwardRef`를 사용

```tsx
import { useRef } from 'react'

const Component = () => {
  const ref = useRef<HTMLInputElement>(null)
  return <MyInput ref={ref} />
}

interface Props {
  name: string
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

### `forwardRef`

#### `forwardRef`의 타입 정의

```tsx
function forwardRef<T, R = {}>(
  render: ForwardRefRenderFunction<T, P>,
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>
```

#### `ForwardRefRenderFunction` 타입 정의

```tsx

interface ForwardRefRenderFunction<T, P = {}> {
	(props: P, ref: ForwardedRef<T>): ReactElement | null
	displayName?: string | undefined
	defaultProps?: never | undefined
	propTypes?: never undefined
}
```

#### `ForwardRef` 타입 정의

- `MutableOjbect` 또는 `RefObject`가 들어올 수 있는 `useRef`의 반환 타입과 달리 ForwardRef에는 오직 MutableRefOjbect만 들어올 수 있음
- `MutableRefObject`가 `RefObject`보다 넓은 범위의 타입을 가지기 때문에, 부모 컴포넌트에서 ref를 어떻게 선언했는지와 관계없이 자식 컴포넌트가 해당 ref를 수용할 수 있음

```tsx
type ForwardRef<T> = ((instance: T) => void) | MutableRefObject<T | null> | null
```

### `useImperativeHandle`

- `forwardRenderFunction`과 함께 쓸 수 있는 훅
- 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스텀 메서드를 호출 가능
- 이에 따라 자식 컴포넌트는 내부 상태나 로직을 관리 → 부모 컴포넌트와의 낮은 결합도

```tsx
// <form> 태그의 submit 함수만 따로 뽑아와서 정의
type CreateFormHandle = Pick<HTMLFormElement, 'submit'>

type CreateFormProps = {
  defaultValues?: CreateFormValue
}

const JobCreateForm: React.ForwardRenderFunction<CreateFormHandle, CreateFormProps> = (
  props,
  ref,
) => {
  // useImperativeHandle 훅을 사용해서 submit 함수 커스터마이징
  useImperativeHandle(ref, () => {
    submit: () => {}
  })
}
```

- 자식 컴포넌트 → ref와 정의된 `CreateFormHandle`을 통해 부모 컴포넌트에서 호출할 수 있는 함수 생성
- 부모 컴포넌트 → `current.submit()`을 사용해서 자식 컴포넌트의 특정 메서드 실행

```tsx
const CreatePage: React.FC = () => {
  // CreateFormHandle 형태를 가진 자식의 ref를 불러옴
  const refForm = useRef<CreateFormHandle>(null)

  const handleSubmitButtonClick = () => {
    // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근 가능
    refForm.current?.submit()
  }
}
```

### `useRef`의 여러 가지 특성

- `useRef`로 관리되는 변수는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않음
- `useRef`로 관리되는 변수는 값을 설정한 후 즉시 조회 가능

> 렌더링에 영향을 미치지 않는 값이며, 값이 변경될 때 다시 렌더링을 기다릴 필요가 없는 경우 사용하면 좋다.

```tsx
type BannerProps = {
  autoplay: boolean
}

const Banner: React.FC<BannerProps> = ({ autoplay }) => {
  const isAutoPlayPause = useRef(false)

  if (autoplay) {
    const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current
  }

  return (
    <>
      {autoplay && (
        <button
          aria-label='자동 재생 일시정지'
          onClick={() => {
            isAutoPlayPause.current = true
          }}
        />
      )}
    </>
  )
}
```

> [!NOTE]
>
> #### 훅의 규칙
>
> - 훅은 항상 최상위 레벨에서 호출되어야 함
> - 훅은 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 함

## 2. 커스텀 훅

- `useInput`와 `onChange`의 인자 타입을 지정하지 않으면 타입스크립트 에러 발생
  - `initialValue` → `string` 타입 정의
  - 이벤트 객체 `e` → `ChangeEvent<HTMLInputElement>` 타입 정의

```tsx
import { useState, useCallback } from 'react'

const useInput = (initialValue: string) => {
  const [value, setValue] = useState(initialValue)

  const onChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value)
  }, [])

  return { value, onChange }
}

export default useInput
```
