## 상태 관리

### 상태(State)

- 리액트에서의 상태: 시간이 지나면서 변할 수 있는 동적인 데이터 → 값이 변경될 때마다 컴포넌트의 렌더링 결과물에 영향
- 지역 상태, 전역 상태, 서버 상태로 분류
- 성능 문제와 상태의 복잡성으로 인해 Redux, MobX, Recoil 같은 외부 상태 관리 라이브러리르 주로 활용

#### 지역 상태(Local State)

- 체크박스의 체크 여부, 폼의 입력값 등 컴포넌트 내부에서 사용되는 상태
- 주로 `useState`으로 생성 및 관리하며 때에 따라 `useReducer`을 사용

#### 전역 상태(Global State)

- 앱 전체에서 공유되는 상태
- 여러 개의 컴포넌트가 전역 상태를 사용할 수도 있으며, 상태가 변경되면 컴포넌트들도 업데이트
- Prop drilling 문제를 피하고자 지역 상태를 해당 컴포넌트들 사이의 전역 상태로 공유할 수도 있음

#### 서버 상태(Server State)

- 사용자 정보, 글 목록 등 외부 서버에 저장해야 하는 상태
- UI 상태와 결합하여 관리하며 로딩 여부나 에러 상태 등을 포함
- react-query, SWR과 같은 외부 라이브러리를 사용하여 관리하기도 함

## 상태를 잘 관리하기 위한 가이드

### 시간이 지나도 변하지 않는다면 상태가 아니다

- 시간이 지나도 변하지 않는 값이라면, 객체 참조 동일성을 유지하는 방법 고려
- 컴포넌트가 마운트될 때만 스토어 객체 인스턴스를 생성하고 컴포넌트가 언마운트될 때까지 해당 참조가 변하지 않는다고 가정

#### 상수 변수에 저장하여 사용

- 렌더링 때마다 새로운 객체 인스턴스 생성 → 매번 다른 객체로 인식 → 불필요한 리렌더링 발생

```tsx
const Component: React.VFC = () => {
	const store = new Store()

	return (
		<StoreProvider store={store}>
			<Children>
		</StoreProvider>
	)
}
```

> 리액트의 다른 기능을 활용하여 컴포넌트 라이프사이클 내에서 마운트될 때 인스턴스가 생성되고, 렌더링될 때마다 동일한 객체 참조가 유지되도록 구현해줘야 한다.

#### `useMemo` 사용

- 메모이제이션: 컴포넌트가 마운트될 때만 객체 인스턴스 생성 → 이후 렌더링은 이전 인스턴스 재활용

```tsx
const store = useMemo(() => new Store(), [])
```

- 객체 참조 동일성을 유지하기 위해 `useMemo`를 사용하는 것은 권장되는 방법이 아님 → 오로지 성능 향상을 위한 용도로만!
- 예상하지 못한 에러 발생 가능
  - 의미상으로 보장되지 않은 메모이제이션
  - 리액트에서 메모리 확보를 위해 이전 메모이제이션 데이터 삭제 가능성
- `useMemo` 없이도 올바르게 동작하도록 코드 작성 → 나중에 성능 개선을 위해 `useMemo` 추가

#### `useState` 사용

- `useState`로 state 생성 시 초기 값만 지정하여 모든 렌더링 과정에서 객체 참조를 동일하게 유지할 수 있음
  - `useState(new Store())` → 객체 인스턴스가 실제로 사용되지 않더라도 렌더링마다 생성되어 초기 값 설정에 큰 비용
  - `useState(() ⇒ new Store())` → 초기 값을 계산하는 콜백을 지정하는 방식 사용(지연 초기화 방식)

```tsx
const [store] = useState(() => new Store())
```

- 기술적으로 잘 동작하지만 의미론적으로 좋은 방법은 아님 → `useState`로 상태를 생성 및 관리하는 목적과 다름
  - 상태란? 시간이 지나면서 변화되어 렌더링에 영향을 주는 데이터
  - 현재의 목적: 모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 함 → 정의한 의미와 맞지 않음

> `useRef()`와 `{ current: … }` 객체를 직접 생성하는 방법 간의 유일한 차이는 useRef는 매번 렌더링할 때마다 동일한 ref 객체를 제공한다는 것이다.

#### `useRef` 사용

- 동일한 객체 참조를 유지하려는 목적으로 가장 적합한 훅
- `useRef(new Store())`를 사용하면 `useState`와 마찬가지로 렌더링마다 불필요한 인스턴스 생성

```tsx
const store = useRef<Store>(null) // 초기 값 null로 지정

if (!store.current) {
  store.current = new Store() // curret 값 확인 후 인스턴스를 current에 할당
}
```

- 가독성 등의 이유로 팀 내에서 합의된 컨벤션으로 지정된 것이 아니라면 동일한 객체 참조를 할 때는 `useRef` 사용을 권장

### 파생된 값은 상태가 아니다

- 부모에게서 전달받을 수 있는 props이거나 기존 상태에서 계산될 수 있는 값 → 상태 X
- SSOT(Single Source Of Truth): 어떠한 데이터도 단 하나의 출처에서 생성하고 수정해야 한다는 원칙을 의미하는 방법론
  - 리액트 앱에서 상태를 정의할 때도 이러한 방법론을 고려해야 함
- 다른 값에서 파생된 값을 상태로 관리하게 되면 기존 출처와는 다른 새로운 출처에서 관리하게 되는 것
  - 데이터의 정확성과 일관성을 보장하기 어려움

#### 처음에는 prop으로 받은 값을 보여주고 이후에는 사용자 입력 값을 보여주는 이메일 입력 폼

```tsx
import { useState } from 'react'

type UserEmailProps = {
  initialEmail: string
}

const UserEmail = ({ initialEmail }: UserEmailProps) => {
  const [email, setEmail] = useState(initialEmail)

  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value)
  }

  return (
    <div>
      <input type="text" value={email} onChange={onChagneEmail} />
    </div>
  )
}
```

- 문제점: `initialEmail` prop의 값이 변경되어도 input 태그의 `value`는 변경되지 않음
- 이유: `useState`의 초기 값으로 설정된 값을 컴포넌트가 마운트될 때 한 번만 email 상태의 값으로 설정되며, 이후에는 독자적으로 관리

#### 🤔 `useEffect`를 사용하여 두 출처를 동기화하기

- `intialEmail` 값이 변경될 때마다 `useEffect`를 실행하여 props와 상태를 동기화

```tsx
import { useState, useEffect } from 'react'

type UserEmailProps = {
  initialEmail: string
}

const UserEmail = ({ initialEmail }: UserEmailProps) => {
  const [email, setEmail] = useState(initialEmail)

  useEffect(() => {
    setEmail(initialEmail)
  }, [initialEmail])

  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value)
  }

  return (
    <div>
      <input type="text" value={email} onChange={onChagneEmail} />
    </div>
  )
}
```

- 문제점: 사용자 입력 값이 변경된 뒤에 `initialEmail`이 변경된다면 사용자의 입력을 무시하고 `intialEmail` 값으로 `value`를 설정
- `useEffect`를 사용한 동기화 작업은 `localStorage` 같은 **리액트 외부 데이터와 동기화**할 때만 사용해야 함
- 내부에 존재하는 상태를 `useEffect`로 동기화하면 개발자가 추적하기 어려운 오류가 발생할 수 있기 때문

#### 🤓 상태 끌어올리기(Lifting State Up)로 하나의 출처로 만들기

- 현재 `email` 상태에 대한 출처 → 2개의 출처
  - prop으로 받는 `initialEmail`
  - `useState`로 생성한 `email`
- 출처를 하나로! → 두 출처 간의 데이터를 동기화하기보다 **단일한 출처**에서 데이터를 사용하도록 변경
- `UserEmail`에서 관리하던 상태를 부모 컴포넌트로 옮겨서 `email` 데이터의 출처를 **props 하나로 통일**

```tsx
import { useState } from 'react'

type UserEmailProps = {
  email: string
  setEmail: React.Dispatch<React.SetStateAction<string>>
}

const UserEmail = ({ email, setEmail }: UserEmailProps) => {
  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value)
  }

  return (
    <div>
      <input type="text" value={email} onChange={onChagneEmail} />
    </div>
  )
}
```

#### 아이템 목록이 변경될 때마다 선택된 아이템 가져오기

- 두 컴포넌트에서 동일한 데이터를 상태로 갖고 있는 경우 **가까운 공통 부모 컴포넌트로 상태를 끌어올려서** SSOT 원칙을 지킬 수 있도록 해야 함

```tsx
const [item, setItems] = useState<Item[]>([])
const [selectedItems, setSelectedItems] = useState<Item[]>([])

useEffect(() => {
  setSelectedItems(item.filter(item => item.isSelected))
}, [])
```

- 문제: `items`와 `selectedItems`가 동기화되지 않을 수 있음
- 여러 상태가 복잡하게 얽혀 있으면 흐름을 파악하기 어렵고 의도치 않게 동기화 과정이 누락될 수도 있음
- setSelectedItems를 사용하여 items에서 가져온 데이터가 아닌 임의의 데이터셋을 설정하는 것도 구조적으로 가능하기 때문에 오류가 발생할 가능성 존재
- 새로운 상태로 정의함으로써 단일 출처가 아닌 여러 출처를 가지게 되었고 이에 따라 동기화 문제가 발생

#### 상태로 정의하지 않고 계산된 값을 자바스크립트 변수로 담기

- 계산으로 변경한 `selectedItems`: `items` 변경 -> 컴포넌트 렌더링 → 다시 계산

```tsx
const [items, setItems] = useState<Item[]>([])
const selectedItems = items.filter(item => item.isSelected) // 계산
```

- `items`, `selectedItems`를 `useEffect`로 동기화
  - `items`의 값이 바뀌며 렌더링 발생
  - `items`의 값이 변경됨을 감지하고 `selectedItems` 값을 변경하며 리렌더링 발생
- `items`는 상태로, `selectedItems`는 계산으로 만들기
  - `items`의 값이 바뀌며 렌더링 발생 O / `selectedItems`가 변경된다고 리렌더링 발생 X
  - 렌더링될 때마다 계산 수행 → 계산 비용이 크다면 성능 문제가 발생할 수 있음 → 그렇다면 `useMemo`로 메모이제이션!

```tsx
const [items, setItems] = useState<Item[]>([])
const selectedItems = useMemo(() => veryExpensiveCalculation(items), [items]) // items가 변경될 때만 계산
```

### `useState` vs `useReducer`

- `useState` 대신 `useReducer` 사용을 권장하는 경우
  - 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직을 다룰 때
  - 다음 상태가 이전 상태에 의존적일 때

```tsx
type DateRangePreset = 'TODAY' | 'LAST_WEEK' | 'LAST_MONTH'

type ReviewRatingString = '1' | '2' | '3' | '4' | '5'

interface ReviewFilter {
  // 리뷰 날짜 필터링
  startDate: Date
  endDate: Date
  dateRangePreset: Nullable<DateRangePreset>
  // 키워드 필터링
  keywords: string[]
  // 리뷰 점수 필터링
  ratings: ReviewRatingString[]

  // ...
}

// Review List Query State
interface State {
  filter: ReviewFilter
  page: string
  size: number
}
```

- 데이터 구조를 useState로 다루면 상태를 업데이트할 때마다 잠재적인 오류 가능성 증가
  - 페이지값만 업데이트하고 싶어도 우선 전체 데이터를 가지고 온 다음 페이지값을 덮어쓰게 됨
  - 사이즈, 필터 같은 업데이트하려는 필드 외의 다른 필드가 수정될 수 있음
- ‘사이즈 필드를 업데이트할 때는 페이지 필드를 0으로 설정해야 한다’ 등의 특정한 업데이트 규칙이 있다면 `useState`만으로는 한계
- `useReducer`는 **무엇을 변경할지**와 **어떻게 변경할지**를 분리하여 `dispatch`를 통해 어떤 작업을 할지를 액션으로 넘기고 `reducer` 함수 내에서 상태를 업데이트하는 방식을 정의하는 훅
- 복잡한 상태 로직을 숨기고 안정성을 높일 수 있음

#### `reducer`로 리뷰 쿼리 상태 정의하여 `useReducer`에서 사용하기

```tsx
// Action 정의
type Action =
	| { payload: ReviewFilter; type: 'filter' }
	| { payload: number; type: 'navigate' }
	| { payload: number; type: 'resize' }

// Reducer 정의
const reducer: React.Reducer<State, Action> = (state, action) => {
	switch (action.type) {
		case 'filter':
			return {
				filter: action.payload,
				page: 0,
				size: state.size
			}
		case 'navigate':
			return {
				filter: state.filter,
				page: action.payload,
				size: state.size
			}
		case 'resize':
			return {
				filter: state.filter,
				page: 0,
				size: action.payload
			}
		default
			return state
	}
}

// useReducer 사용
const [state, dispatch] = useReducer(reducer, getDefaultState())

dispatch({ payload: filter, type: 'filter' })
dispatch({ payload: page, type: 'navigate' })
dispatch({ payload: size, type: 'resize' })
```

#### 불리언 상태를 토글하는 액션만 사용하는 경우에 `useReducer` 사용하기

```tsx
const [fold, setFold] = useState(true)

const toggleFold = () => {
  setFold(prev => !prev)
}

const [fold, toggleFole] = useReducer(v => !v, true)
```

> `reducer` 로직을 따로 추출하면 다음과 같다.

```tsx
const reducer = (state: boolean): boolean => !state
```

## 전역 상태 관리

> [!NOTE]
>
> 상태는 허용하는 곳과 최대한 가까워야 하며 사용범위를 제한해야만 한다.

#### 상태를 다른 컴포넌트와 공유할 수 있는 전역 상태로 사용하는 방법

- 컨텍스트 API + `useState` 또는 `useReducer`
- 외부 상태 관리 라이브러리(Recux, MobX, Recoil 등)

### 컨텍스트 API(Context API)

- 다른 컴포넌트들과 데이터를 쉽게 공유하기 위한 목적으로 제공되는 API
- Prop Drilling 같은 문제를 해결하기 위한 도구로 활용
- 전역적으로 공유해야 하는 데이터를 컨텍스트로 제공 + 해당 컨텍스트를 컴포넌트에서 구독
- UI 테마 정보, 로케일 데이터 등 전역적으로 제공하거나 컴포넌트의 props를 하위 컴포넌트에게 계속해서 전달해야 할 떄 유용하게 사용

#### 유틸리티 함수 createContext

- 프로바이더와 해당 컨텍스를 사용하는 훅을 간편하게 생성을 위한 유틸리티 함수

```tsx
import React from 'react'

type Nullable<T> = T | null
type Consumer<C> = () => C

export interface ContextInterface<S> {
  state: S
}

export function createContext<S, C = ContextInterface<S>>(): readonly [
  React.FC<React.PropsWithChildren<C>>,
  Consumer<C>,
] {
  const context = React.createContext<Nullable<C>>(null)

  const Provider: React.FC<React.PropsWithChildren<C>> = ({ children, ...otherProps }) => {
    return <context.Provider value={otherProps as C}>{children}</context.Provider>
  }

  const useContext: Consumer<C> = () => {
    const _context = React.useContext(context)
    if (!_context) {
      throw new Error('not found context')
    }
    return _context
  }

  return [Provider, useContext]
}
```

- 컨텍스트 API는 엄밀하게 말해 전역 상태를 관리하기 위한 솔루션이라기보다 여러 컴포넌트 간에 값을 공유하는 솔루션에 가까움
- `useState`나 `useReducer` 같이 지역 상태를 관리하기 위한 API와 결합하여 여러 컴포넌트 사이에서 상태를 공유하기 위한 방법으로 사용되기도 함

```tsx
function App() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <StateProvider.Provider value={{ state, dispatch }}>
      <ComponentA />
      <ComponentB />
    </StateProvider.Provider>
  )
}
```

- 해당 컨텍스트를 구독하는 컴포넌트에서 앱에 정의된 상태를 읽고 업데이트 가능
- 그러나 컨텍스트 API를 사용하여 전역 상태를 관리하는 것은 대규모 애플리케이션이나 성능이 중요한 애플리케이션에서 권장되지 않는 방법
- 컨텍스트 프로바이더의 props로 주입된 값이나 참조가 변경될 때ㅏㅁ다 해당 컨텍스트를 구독하고 있는 모든 컴포넌트가 리렌더링되기 때문
- 애플리케이션이 커지고 전역 상태가 많아질 수록 불필요한 리렌더링 발생 및 상태의 복잡도 증가

## 상태 관리 라이브러리

### MobX

- 객체 지향 프로그래밍과 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리
- 상태 변경 로직을 단순하게 작성 가능
- 복잡한 업데이트 로직을 라이브러리에 위임 가능
- 객체 지향 스타일로 코드를 작성하는 데 익숙한 개발자에게 추천
- 데이터가 언제, 어떻게 변하는지 추적하기 어렵기 때문에 트러블슈팅이 어려움

```jsx
import { observer } from 'mobx-react-lite'
import { makeAutoObservable } from 'mobx'

class Cart {
  itemAmout = 0

  constructor() {
    makeAutoObservable(this)
  }

  increase() {
    this.itemAmount += 1
  }

  reset() {
    this.itemAmount = 0
  }
}

const myCart = new Cart()

const CartView = observer(({ cart }) => (
  <button onClick={() => cart.reset()}>amount of cart items: {cart.itemAmount}</button>
))

ReactDom.render(<CartView cart={myCart} />, document.body)

setInterval(() => {
  myCart.increase()
}, 1000)
```

### Redux

- 함수형 프로그래밍의 영향을 받은 라이브러리

#### 장점

- 특정 UI 프레임워크에 종속되지 않아 독립적으로 상태 관리 라이브러리 사용 가능
- 오랜 기간 사용되어 왔기 때문에 다양한 요구 사항에 대해 충분히 검증됨
- 상태 변경 추적에 최적화되어 있어, 특정 상황에서 발생한 애플리케이션 문제의 원인 파악 용이

#### 단점

- 많은 보일러플레이트
- 높은 러닝 커브

```jsx
import { createStore } from 'redux'

function counter(state = 0, action) {
  switch (action.type) {
    case 'PLUS':
      return state + 1
    case 'MINUS':
      return state - 1
    default:
      return state
  }
}

let store = createStore(counter)
store.subscribe(() => console.log(store.getState()))

store.dispatch({ type: 'PLUS' }) // 1
store.dispatch({ type: 'PLUS' }) // 2
store.dispatch({ type: 'MINUS' }) // 1
```

### Recoil

- 상태를 저장하는 Atom과 상태를 변경하는 순수함수 seletor로 상태를 관리하는 라이브러리

#### 장점

- 리덕스에 비해 상대적으로 적은 보일러플레이트
- 비교적 낮은 러닝 커브

#### 단점

- ……현재 개발중단

```jsx
import { RecoilRoot } from 'recoil'
import { TextInput } from './'

function App() {
  return (
    <RecoilRoot>
      <TextInput />
    </RecoilRoot>
  )
}
```

```jsx
import { atom } from 'recoil'

export const textState = atom({
  key: 'textState',
  default: '',
})
```

```jsx
import { useRecoilState } from 'recoil'
import { textState } from './'

export function TextInput() {
  const [text, setText] = useRecoilState(textState)

  const onChange = evenet => {
    setText(evenet.target.value)
  }

  return (
    <div>
      <input type="text" value={text} onChange={onChange} />
      <br />
      Echo: {text}
    </div>
  )
}
```

### Zustand

- flux 패턴 사용
- 많은 보일러플레이트를 가지지 않는 훅 기반의 편리한 API 모듈 제공
- 클로저를 활용하여 스토어 내부 상태 관리하여 특정 라이브러리에 종속되지 않음

```jsx
import { creat } from 'zustand'

const useBearStore = create(set => ({
  bears: 0,
  increasePopulation: () => set(state => ({ bears: state.bears + 1 })),
  removeAllBears: () => set(state => ({ bears: 0 })),
})) // 상태와 상태를 변경하는 액션 정의

function BearCounter() {
  const bears = useBearStore(state => state.bears) // 상태 사용
  return <h1>{bears} around here ... </h1>
}

function Controls() {
  const increasePopulation = useBearStore(state => state.increasePopulation) // 상태 사용
  return <button onClick={increasePopulation}>Plus</button>
}
```
