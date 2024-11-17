## 10장 상태관리
### 10.1 상태 관리
#### 1. 상태
- 리액트 애플리케이션에서의 상태는 렌더링에 영향을 줄 수 있는 동적인 데이터 값
- 공식 문서에서는 렌더링 결과에 영향을 주는 정보를 담은 순수 자바스크립트 객체라고 함

**지역 상태**
- 컴포넌트 내부에서 사용되는 상태

**전역 상태**
- 앱 전체에서 공유하는 상태
- props drilling 문제를 해결하기 위해 사용하기도 함

**전역 상태**
- 사용자 정보, 글 목록 등 외부 서버에 저장해야 하는 상태들
- UI 상태와 결합하여 관리하게 되며 로딩 여부나 에러 상태등을 포함

#### 2. 상태를 잘 관리하기 위한 가이드
- 상태는 애플리케이션의 복잡성을 증가시키고 동작을 예측하기 어렵게 만든다. 또한 상태가 업데이트될때마다 리렌더링이 발생하므로 상태의 개수를 최소화하는 것이 바람직
- 가능하다면 상태가 없는 stateless 컴포넌트 활용
- 어떤 값을 상태로 정의할때는 다음 2가지 사항 고려
  - 시간이 지나도 변하지 않는다면 상태가 아님
  - 파생된 값은 상태가 아님

**시간이 지나도 변하지 않는 값**
- 시간이 지나도 변하지 않는 값이라면 객체 참조 동일성을 유지하는 방법 고려해볼 수 있음
- 단순히 상수 변수에 저장하여 사용할 수 있지만 렌더링될때마다 새로운 객체가 생성되므로 컨텍스트나 props 등으로 전달했을 시 불필요한 리렌더링이 자주 발생할 수 있음
- useMemo를 사용할 수도 있지만 성능 향상을 위한 용도로만 사용해야 함
- useState의 초깃값만 지정하거나 useRef를 사용하는 방법이 있음
```ts
const [store] = useState(()=>new Store());
```
```ts
const store = useRef<Store>(null);
if(!store.current) store.current = new Store();
```

**파생된 값은 상태가 아니다**
- 다른값에서 파생된 값을 상태로 관리하게 되면 기존 출처와는 다른 새로운 출처에서 관리하게 되는 것
- 부모에게서 Props로 전달받으면 상태가 아님

```tsx
import React, { useState } from 'react';

type UserEmailProps = {
  initialEmail: string;
};

const UserEmail: React.VFC<UserEmailProps> = ({ initialEmail }) => {
  const [email, setEmail] = useState(initialEmail); 
  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value);
  };
  return (
    <div>
      <input type='text' value={email} onChange={onChangeEmail} />
    </div>
  );
};
```

- 위 코드의 문제점
  - 컴포넌트가 마운트될때 한번만 email의 값이 초기화되고 이후에는 독자적으로 관리됨
  - useEffect를 사용할 수도 있겠지만 사용자가 값을 입력하더라도 부모로부터 전달된 값을 value로 설정하게 될 것이라 맞지 않음
- 해결방법
  - UserEmail에서 관리하던 상태를 부모 컴포넌트로 옮겨서 email 데이터의 출처를 props 하나로 통일할 수 있음 (Single source of truth)
```tsx
import React, { useState } from 'react';

type UserEmailProps = {
  email: string;
  setEmail: React.Dispatch<React.SetStateAction<string>>;
};

const UserEmail: React.VFC<UserEmailProps> = ({ email, setEmail }) => {
  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value);
  };
  return (
          <div>
            <input type='text' value={email} onChange={onChangeEmail} />
          </div>
  );
};  
```

- 새로운 예시
```tsx
import {useState, useEffect} from 'react';

const [items, setItems] = useState<Item[]>([]);
const [selectedItems, setSelectedItems] = useState<Item[]>([]);

useEffect(() => {
  setSelectedItems(items.filter((item) = > item.isSelected));
}, [items]);
```
- 위 코드의 문제점은 items와 selectedItems가 동기화되지 않을 수 있다는 것
- 아래처럼 변경

```tsx
import {useState} from 'react';

const [items, setItems] = useState<Item[]>([]);
const selectedItems = items.filter((item) = > item.isSelected);
```

- 이제 동기화는되었지만 렌더링될때마다 selectedItems 계산이 수행되므로 만약 계산 비용이 크다면 성능상 문제가 발생할수도 있음

```tsx
import {useState, useMemo} from 'react';

const [items, setItems] = useState<Item[]>([]);
const selectedItems = useMemo(() => veryExpensiveCalculation(items), [items]);
```

- useMemo를 사용하면 items가 변경될때만 selectedItems가 다시 계산되므로 성능상 이점을 얻을 수 있음

**useState vs useReducer 어떤 것을 사용해야 할까**
- useState 대신 useReducer 사용을 권장하는 경우
  - 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직을 다룰 때
  - 다음 상태가 이전 상태에 의존적일 때
- 리스트를 필터링하여 보여주기 위한 쿼리를 상태로 저장해야 한다면?

```ts
// 날짜 범위 기준 - 오늘, 1주일, 1개월
type DateRangePreset = 'TODAY' | 'LAST_WEEK' | 'LAST_MONTH';
type ReviewRatingString = '1' | '2' | '3' | '4' | '5';
interface ReviewFilter {
// 리뷰 날짜 필터링
startDate: Date;
endDate: Date;
dateRangePreset: Nullable<DateRangePreset>;
// 키워드 필터링
keywords: string[];
// 리뷰 점수 필터링
ratings: ReviewRatingString[];
// ... 이외 기타 필터링 옵션
}
// Review List Query State
interface State {
filter: ReviewFilter;
page: string;
size: number;
}
```

- 이러한 데이터 구조를 useState로 다루면 상태를 업데이트할때마다 잠재적인 오류 가능성 증가
- 페이지 값만 업데이트하고 싶어도 우선 전체 데이터를 가지고 온 다음 페이지값을 덮어쓰게 되므로 사이즈나 필터같은 다른 필드가 수정될 수 있어 의도치 않은 오류 발생
- 사이즈 필터를 업데이트할 때는 페이지 필드를 0으로 설정해야 한다 등의 업데이트 규칙이 있다면 useState만으로는 한계가 있음. 이럴 때는 useReducer를 사용하는 것이 좋음
- useReducer는 무엇을 변경할지와 어떻게 변경할지를 분리하여 dispatch를 통해 어떤 작업을 할지를 액션으로 넘기고 reducer 함수 내에서 상태를 업데이트하는 방식을 정의

```tsx
import React, { useReducer } from 'react';

// Action 정의
type Action =
| { payload: ReviewFilter; type: 'filter'; }
| { payload: number; type: 'navigate'; }
| { payload: number; type: 'resize'; };
// Reducer 정의
const reducer: React.Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    case 'filter':
      return {
        filter: action.payload,
        page: 0,
        size: state.size,
      };
    case 'navigate':
      return {
        filter: state.filter,
        page: action.payload,
        size: state.size,
      };
    case 'resize':
      return {
        filter: state.filter,
        page: 0,
        size: action.payload,
      };
    default:
      return state;
    }
};

// useReducer 사용
const [state, dispatch] = useReducer(reducer, getDefaultState());
// dispatch 예시
dispatch({ payload: filter, type: 'filter' });
dispatch({ payload: page, type: 'navigate' });
dispatch({ payload: size, type: 'resize' });
```

- 이외에도 boolean 상태를 토글하는 액션만 사용하는 경우에는 useState 대신 useReducer를 사용하곤 함

```ts
import { useReducer } from 'react';

//Before
const [fold, setFold] = useState(true);

const toggleFold = () => {
  setFold((prev) => !prev);
};

// After
const [fold, toggleFold] = useReducer((v) => !v, true);
```

#### 3. 전역 상태 관리와 상태 관리 라이브러리
- 상태는 사용하는 곳과 최대한 가까워야 하며 사용 범위를 제한해야만 한다.
- 상태를 전역으로 사용하는 방법은 크게 두가지로 나눌 수 있다
  - 컨텍스트 API + useState 또는 useReducer
  - 외부 상태 관리 라이브러리(Redux, MobX, Recoil 등)

**컨텍스트 API(Context API)**
```tsx
// 현재 구현된 것 - TabGroup 컴포넌트뿐 아니라 모든 Tab 컴포넌트에도 type prop을 전달
<TabGroup type='sub'>
  <Tab name='텝 레이블 1' type='sub'>
    <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2' type='sub'>
    <div>123</div>
  </Tab>
</TabGroup>
// 원하는 것 - TabGroup 컴포넌트에만 전달
<TabGroup type='sub'>
  <Tab name='텝 레이블 1'>
   <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2'>
    <div>123</div>
  </Tab>
</TabGroup>
```
- 위와 같은 상황에서는 컨텍스트 API를 사용하여 전역 상태를 관리하는 것이 좋음
```tsx
import {FC} from 'react';

const TabGroup: FC<TabGroupProps> = (props) => {
  const { type = 'tab', ...otherProps } = useTabGroupState(props);
  /* ... 로직 생략 */
  return (
    <TabGroupContext.Provider value={{ ...otherProps, type }}>
      {/* ... */}
    </TabGroupContext.Provider>
  );
};


const Tab: FC<TabProps> = ({ children, name }) => {
  const { type, ...otherProps } = useTabGroupContext();
  return <>{/* ... */}</>;
};
```
- 컨텍스트 API와 관련한 또 하나의 팁은 유틸리티 함수를 정의하여 더 간단한 코드로 컨텍스트와 훅을 생성하는 것
```tsx
import React from 'react';

type Consumer<C> = () => C;

export interface ContextInterface<S> {
  state: S;
}

export function createContext<S, C = ContextInterface<S>>(): readonly [React.FC<C>, Consumer<C>] {
  const context = React.createContext<Nullable<C>>(null);

  const Provider: React.FC<C> = ({ children, ...otherProps }) => {
    return (
      <context.Provider value= {otherProps as C}>{children}</context.Provider>
    );
  };

  const useContext: Consumer<C> = () => {
    const _context = React.useContext(context);
    if (!_context) {
      throw new Error(ErrorMessage.NOT_FOUND_CONTEXT);
    }
    return _context;
  };

  return [Provider, useContext];
}

// Example
interface StateInterface {}
const [context, useContext] = createContext<StateInterface>();
```

- 그러나 컨텍스트 API를 사용하여 전역 상태를 관리하는 것은 대규모 애플리케이션이나 성능이 중요한 애플리케이션에서 권장되지 않는 방법
- 관심사를 잘 분리해서 구성하는 것이 중요

### 10.2 상태 관리 라이브러리
#### 1. Mobx
- 객체 지향 프로그래밍과 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리
- 상태 변경 로직을 단순하게 작성할 수 있고, 복잡한 업데이트 로직을 라이브러리에 위임할 수 있음
- 다만 데이터가 언제, 어떻게 변하는지 추적하기 어렵기 때문에 트러블슈팅에 어려움을 겪을 수 있음

```tsx
import { observer } from 'mobx-react-lite';
import { makeAutoObservable } from 'mobx';

class Cart {
  itemAmount = 0;

  constructor() {
    makeAutoObservable(this);
  }

  increase() {
    this.itemAmount += 1;
  }

  reset() {
    this.itemAmount = 0;
  }
}

const myCart = new Cart();
const CartView = observer(({ cart }) => (
  <button onClick={() => cart.reset()}>
    amount of cart items: {cart.itemAmount}
  </button>
));

ReactDOM.render(<CartView cart= {myCart} />, document.body);
```

#### 2. Redux
- 함수형 프로그래밍의 영향을 받은 상태 관리 라이브러리
- 특정 UI 프레임워크에 종속되지 않아 독립적으로 상태 관리 라이브러리를 사용할 수 있음
- 상태 변경 추적에 최적화되어있고 오랜 시간 사용되어왔기 때문에 다양한 요구 사항에 대해 충분히 검증됨
- 다만, 단순한 상태 설정에도 많은 보일러플레이트가 필요하고, 사용 난도가 높다는 단점이 있음
```tsx
import { createStore } from 'redux';

function counter(state = 0, action) {
  switch (action.type) {
    case 'PLUS':
      return state + 1;
    case 'MINUS':
      return state - 1;
    default:
      return state;
  }
}

let store = createStore(counter);

store.subscribe(() => console.log(store.getState()));

store.dispatch({ type: 'PLUS' });
// 1
store.dispatch({ type: 'PLUS' });
// 2
store.dispatch({ type: 'MINUS' });
// 1
```

#### Recoil
- 상태를 저장할 수 있는 Atom과 해당 상태를 변형할 수 있는 순수 함수 selector를 통해 상태를 관리하는 라이브러리
- Redux에 비해 보일러플레이트가 적고 난이도가 쉽지만 아직 실험적인 상태라 충분한 검증 이루어지지 않았음

```tsx
import React from 'react';
import { RecoilRoot } from 'recoil';
import { TextInput } from './';

function App() {
  return (
    <RecoilRoot>
      <TextInput />
    </RecoilRoot>
  );
}

import { atom } from 'recoil';

export const textState = atom({
  key: 'textState', // unique ID (with respect to other atoms/selectors)
  default: '', // default value (aka initial value)
});

import { useRecoilState } from 'recoil';
import { textState } from './';

export function TextInput() {
  const [text, setText] = useRecoilState(textState);
  const onChange = (event) => {
    setText(event.target.value);
  };
  return (
          <div>
            <input type='text' value= {text} onChange= {onChange} />
            <br />
            Echo: {text}
          </div>
  );
}

setInterval(() => {
  myCart.increase();
}, 1000);
```

#### 4. zustand
- Flux 패턴을 사용하며 많은 보일러플레이트를 가지지 않는 훅 기반의 편리한 API들 제공
- 클로저를 활용하여 스토어 내부 상태를 관리함으로써 특정 라이브러리에 종속되지 않는 특징이 있음

```tsx
import { create } from 'zustand';

const useBearStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

function BearCounter() {
  const bears = useBearStore((state) => state.bears);

  return <h1>{bears} around here ...</h1>;
}

function Controls() {
  const increasePopulation = useBearStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>Plus</button>;
}
```
