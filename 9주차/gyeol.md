# 10장 - 상태관리

## 10.1 상태 관리

### 1. 상태(State)

- 리액트 애플리케이션에서의 상태는 렌더링에 영향을 줄 수 있는 동적인 데이터 값을 뜻함
- 리액트 공식문서에서는 상태를 **렌더링 결과에 영향을 주는 정보를 담은 순수 자바스크립트 객체**라고 정의하고 있음
- 리액트 앱 내의 상태는 지역 상태, 전역상태, 서버 상태로 분류 가능

### ✅ 지역 상태(Local State)

- 컴포넌트 내부에서 사용되는 상태
- 주로 `useState` 훅을 사용해서 관리하며, 때에 따라 `useReducer`와 같은 훅을 사용하기도 함
- e.g. 체크박스의 체크 여부나 폼의 입력 값 등

### ✅ 전역 상태(Global State)

- 앱 전체에서 공유하는 상태
- 여러 개의 컴포넌트가 전역 상태를 사용할 수 있고 상태가 변경되면 컴포넌트들도 업데이트 됨
- Prop drilling 문제를 피하기 위해 지역 상태를 해당 컴포넌트들 사이의 전역 상태로 공유할 수도 있음
    
    > **Prop drilling**
    > 
    > props를 통해 데이터를 전달하는 과정에서 중간 컴포넌트는 해당 데이터가 필요하지 않음에도 자식 컴포넌트에 전달하기 위해 props를 전달해야 하는 과정을 뜻함
    > 컴포넌트의 수가 많아지면 Prop drilling으로 인해 코드가 복잡해질 수 있음
    

### ✅ 서버 상태(Server State)

- 사용자 정보, 글 목록 등 외부 서버에 저장해야하는 상태들을 의미함
- UI 상태와 결합하여 관리하게 되며 로딩 여부나 에러 상태 등을 포함
- 지역 상태 또는 전역 상태와 동일한 방법으로 관리되며 최근에는 `react-query`, `SWR`와 같은 외부 라이브러리를 사용하여 관리하기도 함

### 2. 상태를 잘 관리하기 위한 가이드

- 상태는 애플리케이션의 복잡성을 증가시키고 동작을 예측하기 어렵게 만듦
- 상태가 업데이트될 때마다 리렌더링이 발생하므로 유지보수 및 성능 관점에서 상태의 개수를 최소화하는 것이 바람직함
    
    → 가능한 상태가 없는 Stateless 컴포넌트들을 활용하는 것이 좋음
    
- 값을 상태로 정의할 때 고려해야할 것들
    - **시간이 지나도 변하지 않는다면 상태가 아니다**
    - **파생된 값은 상태가 아니다**

### ✅ 시간이 지나도 변하지 않는다면 상태가 아니다

- 시간이 지나도 변하지 않는 값이라면 객체 참조 동일성을 유지하는 방법을 고려해볼 수 있음
- 예시 상황 - 컴포넌트가 마운트될 때만 스토어 객체 인스턴스를 생성하고 컴포넌트가 언마운트될 때까지 해당 참조가 변하지 않는 경우
    - 상수 변수에 저장해서 사용하게 되면 렌더링될 때마다 **새로운 객체 인스턴스가 생성**되기 때문에 컨텍스트나 props 등으로 전달시 매번 다른 객체로 인식되어 불필요한 리렌더링이 자주 발생할 수 있음
    - **리액트의 다른 기능을 활용해서 컴포넌트 라이프사이클 내에서 마운트될 때 인스턴스가 생성되고 렌더링될 때마다 동일한 객체 참조가 유지되도록 구현 필요**
    
    ```tsx
    import React from 'react';
    
    const Component: React.VFC = () => {
      const store = new Store();
      return (
        <StoreProvider store={store}>
         <Children>
        </StoreProvider>
      );
    };
    ```
    
- 객체의 참조 동일성을 유지하기 위해 널리 사용되는 방법 중 하나는 `메모이제이션`
- `useMemo` 를 활용하여 컴포넌트가 마운트될 때만 객체 인스턴스를 생성하고 이후 렌더링에서는 이전 인스턴스를 재활용할 수있도록 구현이 가능하지만 `useMemo` 를 사용하는 것은 바람직하지 않음
    - 객체 참조 동일성을 유지하기 위해 useMemo를 사용하는 것이 권장되지 않는 이유
        - useMemo를 통한 메모이제이션은 의미상으로 보장된 것이 아니므로 성능 향상을 위한 용도로만 사용되어야함
        - 리액트에서는 메모리 확보를 위해 이전 메모이제이션 데이터가 삭제될 수 있음
            
            → useMemo는 성능 개선을 위해 추가하는 것이 바람직함
            
- **객체 참조 동일성을 유지하기 위해 시도할 수 있는 방법**
    - **useState의 초깃값만 지정하는 방법**
        - `useState(new Store());` 와 같이 사용하면 실제로 사용하지 않더라도 객체 인스턴스가  렌더링마다 생성되어 초깃값 설정에 큰 비용이 소요될 수 있음
        **→ `useState(()=> new Store())` 와 같이 초깃값을 계산하는 콜백을 지정하는 방식(지연 초기화 방식)을 사용해야함**
        - 예시 코드
            
            ```tsx
            import {useState} from 'react'
            
            const [store] = useState(() => new Store());
            ```
            
        - useState를 사용하는 것은 기술적으로 잘 동작할 수는 있지만, 의미론적으로 봤을 때 좋은 방법은 아님
        **→ 원래 상태(State)라는 건 시간이 지나면서 변화되어 렌더링에 영향을 주는 데이터인데, 여기서는 모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 하는 것이 목적이기 때문!**
    - useRef를 사용하는 방법
        - 리액트 공식문서에 따르면 동일한 객체 참조를 유지하려는 목적으로는  useRef이 가장 적합한 훅임
        - useRef의 인자로 직접 new Store()을 사용하면 렌더링마다 불필요한 인스턴스가 생성되므로 아래와 같이 사용해야함
            
            ```tsx
            import {useRef} from 'react';
            
            const store = useRef<Store>(null);
            
            if (!store.current) {
              store.current = new Store();
            }
            ```
            
        - **useRef는 기술적으로 useState({children: initialValue})[0]과 동일하지만** 상태(State)가 가진 의미와 객체 참조 동일성을 유지하기 위해 useState에 초깃값만 할당하는 것은 의미가 맞지 않기 때문에 useState를 사용하는 것은 적절치 않음
        **→ 웬만하면 useRef를 사용하는 것을 권장**

### ✅ 파생된 값은 상태가 아니다

- 부모에게서 전달받을 수 있는 props이거나 기존 상태에서 계산될 수 있는 값은 상태(state)가 아님
- SSOT(Single Source Of Truth)
    - 어떠한 데이터도 단 하나의 출처에서 생성하고 수정해야한다는 원칙을 의미하는 방법론
- 리액트 앱에서 다른 값에서 파생된 값을 상태로 관리하게되면 기존 출처와는 다른 새로운 출처에서 관리하게 되는 것이므로 해당 데이터의 정확성과 일관성을 보장하기 어려움
- 문제 예시 코드 ①
    
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
    
    - 위 코드에서 initialEmail prop의 값이 변경되어도 input 태그의 value는 변경되지 않음
    → useState의 초깃값으로 설정한 값은 컴포넌트가 마운트될 때 한 번만  email의 상태 값으로 설정되어 이후에는 독자적으로 관리되기 때문
    **⇒ email 상태에 대한 출처가 prop으로 받는 initialEmail과 useState로 생성한 email state라 문제가 발생**
    - 이 상황에서 props와 상태를 동기화 하기 위해 useEffect를 사용한다면?
        
        ```tsx
        // useEffect를 사용한 코드 예시
        import {useState, useEffect} from 'react';
        
        const [email, setEmail] = useState(initialEmail);
        
        useEffect(() => {
          setEmail(initialEmail);
        }, [initialEmail]);
        ```
        
        → 만약 사용자가 값을 변경한 후 initialEmail prop의 값이 변경된다면 input 태그의 value는 사용자의 입력을 무시하고 부모 컴포넌트로부터 전달된 initialEmail prop의 값을 value로 설정하게 됨
        
        - **이처럼 useEffect를 사용한 동기화 작업은 리액트 외부 데이터(e.g. LocalStorage)와 동기화할 때만 사용해야함.**
        - 내부에 존재하는 상태를 useEffect로 동기화 할 경우 개발자가 추적하기 어려운 오류가 발생될 수 있어 피해야함
    - 위 문제를 해결하기 위해 두 출처 간의 데이터를 동기화하는 것 보다 단일한 출처에서 데이터를 사용하도록 변경해줘야함
        
        → 상위 컴포넌트에서 상태를 관리하도록 해주는 상태 끌어올리기 기법을 적용
        
        ```tsx
        import React, { useState } from 'react';
        
        // 기존 UserEmail에서 관리하던 상태를 아래와 같이 
        // 부모 컴포넌트로 옮겨서 email 데이터의 출처를 props 하나로 통일
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
        
- 위에 나온 예시들처럼 **두 컴포넌트에서 동일한 데이터를 상태로 갖고 있을 때 두 컴포넌트 간의 상태를 동기화하는 방법 대신 가까운 공통 부모 컴포넌트로 상태를 끌어올려 SSOT를 지킬 수 있도록 해야함**
- 문제 예시 코드 ②
    
    ```tsx
    // 아이템 목록과 선택된 아이템 목록을 가지고 있는 코드
    import {useState, useEffect} from 'react';
    
    const [items, setItems] = useState<Item[]>([]);
    const [selectedItems, setSelectedItems] = useState<Item[]>([]);
    
    useEffect(() => {
      setSelectedItems(items.filter((item) = > item.isSelected));
    }, [items]);
    ```
    
    - 아이템 목록이 변경될 때마다 선택된 아이템 목록을 가져오기 위해 useEffect로 동기화 작업을 하고 있음
    → items와 selectedItems가 동기화되지 않을 가능성이 있어 문제가 될 수 있음. 
    또 setSelected를 사용해서 items에서 가져온 데이터가 아닌 임의의 데이터셋을 설정하는 것도 구조적으로 가능하므로 오류가 발생할 가능성이 존재
    - 즉, 새로운 상태(selectedItems)로 정의함으로써 단일 출처가 아닌 여러 출처를 가지게 되었고, 이로 인해 동기화 문제가 발생함
    → 내부의 상태끼리 동기화하는 방법이 아닌 여러 출처를 하나의 출처로 합치는 방법이 적합
    - 성능 측면에서 봤을 때 items와 selectedItems 2가지 상태를 유지하면서 useEffect로 동기화하는 과정을 거치면 selectedItems의 값을 얻기 위해 두번의 렌더링이 발생하게됨
    (items의 값이 바뀌면 렌더링이 발생, items의 값이 변경됨을 감지하고 selectedItems 값을 변경하며 리렌더링 발생)
    - **여러 출처를 하나의 출처로 합치는 가장 간단한 방법**
        
        **: 상태로 정의하지 않고 계산된 값을 자바스크립트의 변수로 담는 것**
        
        ```tsx
        // 자바스크립트 변수로 처리한 코드
        import {useState} from 'react';
        
        const [items, setItems] = useState<Item[]>([]);
        const selectedItems = items.filter((item) = > item.isSelected);
        ```
        
        - 이 경우 items가 변경될 때마다 컴포넌트가 새로 렌더링되며 매번 렌더링될 때마다 selectedItems를 다시 계산하게됨 
        → **만약 계산 비용이 크다면 성능 문제가 발생할 수 있으므로 useMemo를 사용해서 items가 변경될 때만 계산을 수행하고 결과를 메모이제이션하여 성능 가능**
            
            ```tsx
            // useMemo 적용 코드
            import {useState, useMemo} from 'react';
            
            const [items, setItems] = useState<Item[]>([]);
            const selectedItems = useMemo(()=> veryExpensiveCalculation(items), [items]);
            ```
            

### ✅ useState vs useReducer, 어떤 것을 사용해야 할까

- useState 대신 useReducer 사용을 권장하는 경우
    - **다수의 하위 필드를 포함하고 복잡한 상태 로직을 다룰 때**
    - **다음 상태가 이전 상태에 의존적일 때**
- 문제 상황
    - 배달의 민족 리뷰 리스트를 필터링하여 보여주기 위한 쿼리를 상태로 저장해야하는 경우
    - 쿼리는  검색 날짜 범위, 리뷰 점수, 키워드 등 많은 하위 필드를 가지게됨
    
    ```tsx
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
    
    - 위와 같이 복잡한 데이터 구조를 useState로 다루면 상태 업데이트시 오류가 생길 가능성이 증가하게됨
    - 오류 예시
        - 페이지값만 업데이트 하고 싶지만 전체 데이털르 가져온 후 페이지값이 덮어쓰기되므로 사이즈나 필터 같은 다른 필드가 수정될 수 있음
        - 특정한 업데이트 규칙(e.g. 사이즈 필드 업데이트시 페이지 필드를 0으로 설정해야함)이 있다면 useState로는 한계가 있음
    - 이 경우 useReducer를 사용하는 것이 좋음
    - useReducer
        - 무엇을 변경해야할지와 어떻게 변경해야할지를 분리하여 dispatch를 통해 어떤 작업을 할지 액션으로 넘기고 reducer 함수 내에서 상태를 업데이트 하는 방식을 정의
        - useReducer를 사용하면 복잡한 상태 로직을 숨기고 안정성을 높일 수 있음
        - reducer 함수 내에서는 switch 문을 사용하는 게 규칙임
    - 리뷰 쿼리 상태에 대한 Reducer를 정의하여 useReducer를 사용한 코드
        
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
        
- useState 대신 useReducer를 사용하는 다른 예시
: boolean 상태를 토글하는 액션만 사용하는 경우
    
    ```tsx
    import { useReducer } from 'react';
    
    //Before
    const [fold, setFold] = useState(true);
    
    const toggleFold = () => {
      setFold((prev) => !prev);
    };
    
    // After
    const [fold, toggleFold] = useReducer((v) => !v, true);
    ```
    

### 3. 전역 상태 관리와 상태 관리 라이브러리

> *상태는 사용하는 곳과 최대한 가까워야 하며 사용 범위를 제한해야만 한다.*
> 
- 어떠한 상태를 전역 상태로 사용하는 방법
    - 컨텍스트 API와 useState 또는 useReducer 사용하기
    - 외부 상태 관리 라이브러리(Redux, MobX, Recoil 등) 사용하기

### ✅ 컨텍스트 API(Context API)

- 다른 컴포넌트들과 데이터를 쉽게 공유하기 위한 목적으로 제공되는 API
- 컨텍스트 API를 활용하면 전역적으로 공유해야 하는 데이터를 컨텍스트로 제공하고 해당 컨텍스트를 구독한 컴포넌트에서만 데이터를 읽을 수 있게됨
→ UI 테마 정보나, 로케일 데이터 같이 전역적으로 제공하거나 컴포넌트의 props를 하위 컴포넌트에게 계속해서 전달해야할 때 유용하게 사용 가능
- 문제 상황
: TabGroup 컴포넌트와 Tab 컴포넌트에 type이라는 prop을 전달한 경우, TabGroup 컴포너트에만 이 prop을 전달하고 Tab 컴포넌트의 구현 내에서도 사용할 수 있게 하려면?
    
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
    
- 해결
    
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
    
    - 상위 컴포넌트(TabGroup)의 props를 하위 컴포넌트(Tab)에 편리하게 전달하기 위해서는 상위 컴포너트 구현 부에 컨텍스트 프로바이더를 넣어주고 하위 컴포넌트에서 해당 컨텍스트를 구독하여 데이터를 읽어오는 방식을 사용할 수 있음
- 컨텍스트 API 사용시 유틸리티 함수를 정의하여 더 간단한 코드로 컨텍스트와 훅을 생성할 수 있음
    
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
    
    - createContext라는 유틸리티 함수를 정의해서 자주 사용되는 프로바이더와 해당 컨텍스트를 사용하는 훅을 간편하게 생성
- 컨텍스트 API는 전역 상태를 관리하기 위한 솔루션이라기 보다는 여러 컴포넌트 간에 값을 공유하는 솔루션에 가까움
- useState나 useReducer 같이 지역 상태를 관리하기 위한 API와 결합하여 여러 컴포넌트 사이에서 상태를 공유하기 위한 방법으로 사용되기도 함
    
    ```tsx
    // useReducer와 함께 사용
    import { useReducer } from 'react';
    
    function App() {
      const [state, dispatch] = useReducer(reducer, initialState);
      return (
        <StateProvider.Provider value={{ state, dispatch }}>
          <ComponentA />
          <ComponentB />
        </StateProvider.Provider>
      );
    }
    ```
    
    - 위처럼 사용시 해당 컨텍스트를 구독하는 컴포넌트에서 앱에 정의된 상태를 읽고 업데이트가 가능
    - **하지만 컨텍스트 API를 사용해서 전역 상태를 관리하는 것은 대규모 애플리케이션이나 성능이 중요한 애플리케이션에서는 권장되지 않음**
    → 컨텍스트 프로바이더의 props로 중비된 값이나 참조가 변경될 때마다 해당 컨텍스트를 구독하고 있는 모든 컴포넌트가 리렌더링됨
        
        ⇒ 불필요한 리렌더링과 상태의 복잡성이 증가할 수 있음
        

## 2. 상태 관리 라이브러리

### 1. MobX

- 객체 지향 프로그래밍와 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리
- 상태 변경 로직을 단순하게 작성이 가능, 복잡한 업데이트 로직을 라이브러리에 위임 가능
- 객체 지향 스타일로 코드를 작성하는 것에 익숙하다면 추천
- 데이터가 언제, 어떻게 변하는지 추적이 어렵다는 것이 단점
- 예시 코드
    
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
    

### 2. Redux

- 함수형 프로그래밍의 영향을 받은 라이브러리
- 특정 UI 프레임워크에 종속되지 않아 독립적으로 상태 관리 라이브러리를 사용 가능함
- 오랜 기간 사용되어옴 → 다양한 요구사항에 대해 충분히 검증
- 상태 변경 추척에 최적화 되어있음
- 단순한 상태 설정에도 많은 보일러플레이트가 필요하고 러닝커브가 있다는 것이 단점
- 예시 코드
    
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
    

### 4. Zustand

- Flux 패턴을 사용하여 많은 보일러플레이트를 가지지 않는 훅 기반의 편리한 API 모듈을 제공
- 클로저를 활용하여 스토어 내부 상태를 관리함으로써 특정 라이브러리에 종속되지 않는 것이 특징
- 상태와 상태를 변경하는 액션을 정의하고 반환된 훅을 어느 컴포너트에서나 임포트하여 원하는 대로 사용 가능
- 예시 코드
