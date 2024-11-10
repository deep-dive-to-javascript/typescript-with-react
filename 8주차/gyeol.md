# 9장 - 훅

## 9.1 리액트 훅

- 리액트에 훅이 추가되기 전에는(리액트 16.8 버전 이전) 클래스 컴포넌트에서만 상태를 가질 수 있었음
- 클래스 컴포넌트에서는 `componentDidMount`, `componentDidUpdate` 와 같이 하나의 생명주기 함수에서만 상태 업데이트에 따른 로직을 실행시킬 수 있었음
→ 프로젝트의 규모가 커지면서 상태 업데이트, 사이드 이펙트 처리가 불편해짐, 또한 모든 상태를 하나의 함수 내에서 처리하다보니 코드가 복잡해지고 디버깅이 어려워졌음
- 리액트 훅이 도입되면서 함수 컴포넌트에서도 컴포넌트의 생명 주기에 맞춘 로직 실행이 가능해짐

### 1. useState

- 상태를 관리하기 위해 사용하는 훅
- useState의 타입 정의(@types/react v18 기준)
    
    ```tsx
    function useState<S>(
    	initialState: S | (() => S)
    ): [S, Dispatch<SetStateAction<S>>];
    
    type Dispatch<A> = (value: A) => void;
    type SetStateAction<S> = S | ((prevState: S) => S);
    ```
    
    - useState가 반환하는 튜플
        - 첫 번째 요소 : 제네릭으로 지정한 S 타입
        - 두 번째 요소 : 상태를 업데이트할 수 있는 Dispatch 타입의 함수
    - Dispatch 함수의 제네릭으로 지정한 SetStateAction
        - useState로 관리할 상태 타입인 `S` 또는 이전 상태 값을 받아 새로운 상태를 반환하는 함수인 `(prevState: S) => S`가 들어갈 수 있음
- useState에 타입스크립트를 적용하지 않을 때 발생 할 수 있는 문제
    
    ```jsx
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
    
    - 기존 memberList 배열 요소에 없는 agee라는 잘못된 속성이 포함된 객체로 인해 sunAge 변수가 NaN이 되는 예상치 못한 사이드 이펙트가 발생할 수 있음
- 타입스크립트를 사용하면 컴파일타임에  해당 타입 에러를 발견할 수 있음
    
    ```tsx
    import { useState } from "react";
    
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
    

### 2. 의존성 배열을 사용하는 훅

### ✅ useEffect와 useLayoutEffect

### useEffect

- **렌더링 이후 리액트 함수 컴포넌트에 어떤 일을 수행해야 하는지 알려주기 위해 활용 가능**
- useEffect의 타입 정의(@types/react v18 기준)
    
    ```tsx
    type DependencyList = readonly unknown[];
    type EffectCallback = () => void | Destructor;
    function useEffect(effect: EffectCallback, deps?: DependencyList): void;
    ```
    
    - **effect : EffectCallback**
        - useEffect의 첫 번째 인자인 effect의 타입으로 **Destructor를 반환하거나 아무것도 반환하지 않는 함수**
        - Promise 타입은 반환하지 않으므로 **useEffect의 콜백 함수에는 비동기 함수가 들어갈 수 없음**
            
            → useEffect에서 비동기 함수를 호출할 수 있다면 경쟁 상태를 불러일으킬 수 있기 때문
            
            > **경쟁 상태(Race Condition)**
            > 
            > 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제. 이러한 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램 동작이 원하지 않는 방향으로 흐를 수 있음
            > 
    - **deps?: DependencyList**
        - 옵셔널하게 제공되며 effect가 수행되기 위한 조건을 나열
        - **useEffect는 deps가 변경되었는지에 대해 얕은 비교로만 판단**
        → 실제 객체 값이 바뀌지 않았더라도 객체의 참조 값이 변경되면 콜백 함수가 실행되어 원치 않는 렌더링이 반복될 수 있음
            
            ⇒ 이를 방지 하기 위해 실제로 사용하는 값을 useEffect의 deps에서 사용해야함
            
    - useEffect는 Destructor를 반환함(Destructor를 클린업 함수라고도 함)
        - deps가 빈배열이라면 useEffect의 콜백 함수는 컴포넌트가 처음 렌더링될 때만 실행되며 이때의 Destructor는 컴포넌트가 마운트 해제될 때 실행됨
        - deps 배열이 존재한다면 배열의 값이 변경될 때마다 Destructor가 실행됨
            
            > **클린업(Cleanup) 함수**
            > 
            > useEffect나 useLayoutEffect와 같은 리액트 훅에서 사용되며, 컴포넌트가 해제되기 전에 정리(cleanu up) 작업을 수행하기 위한 함수를 말함
            > 

### useLayoutEffect

- useEffect와 비슷한 역할을 하는 훅으로 타입 정의도 동일하지만, 역할의 차이가 존재
    
    ```tsx
    type DependencyList = readonly unknown[];
    function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;
    ```
    
- componentDidUpdate와 같은 기존 생명주기 함수와는 달리 **레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행됨**
- 화면에 해당 컴포넌트가 그려지기 전에 콜백함수를 실행하기 때문에 첫 번째 렌더링 때 빈 이름이 뜨는 경우를 방지할 수 있음

### ✅ useMemo와 useCallback

- 이전에 생성된 값 또는 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 해주는 훅
- 어떤 값을 계산하는 데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용하게 사용할 수 있음
    
    ```tsx
    type DependencyList = readonly unknown[];
    function useMemo<T>(factory: () => T, deps: DependencyList): T;
    function useCallback<T extends Function>(callback: T, deps: DependencyList): T;
    ```
    
    - 두 훅 모두 제네릭을 지원하기 위해 T 타입을 선언.
    - 두 훅 모두 deps 배열을 가지고 있으며, 해당 의존성이 변경되면 값을 다시 계산하게 됨
    → 변경 여부에 대해 얕은 비교로 판단
    - 불필요한 곳에 사용하지 않는 것이 중요함 
    → 모든 값과 함수를 과도하게 메모이제이션하면 컴포넌트의 성능 향상의 보장이 어려울 수 있음

### 3. useRef

- 리액트에서 `<input />` 요소에 포커스를 설정하거나 특정 컴포넌트의 위치로 스크롤을 하는 등 DOM을 직접 선택해야하는 경우에 useRef 훅을 사용할 수 있음
- useRef는 세 종류의 타입 정의를 가지고 있으며, 이는 인자 타입에 따라 반되는 타입이 달라짐
    
    ```tsx
    interface MutableRefObject<T> {
    	current: T;
    }
    interface RefObject<T> {
      readonly current: T | null;
    }
    function useRef<T>(initialValue: T): MutableRefObject<T>;
    function useRef<T>(initialValue: T | null): RefObject<T>;
    function useRef<T = undefined>(): MutableRefObject<T | undefined>;
    ```
    
    - useRef는 MutableRefObject 또는 RefObject를 반환
- MutableRefObject의 current는 값 변경이 가능함
- RefObject의 cureent는 readonly로 값 변경이 불가능
- useRef 예시
    
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
    
    - useRef의 제네릭으로 HTMLInputElement을 넣고, 인자로는 null을 넣었기 때문에
    `function useRef<T>(initialValue: T | null): RefObject<T>;` 이 타입 정의를 따르게 됨
    → 위 코드의 useRef 훅은 RefObject를 반환하여 ref.current 값을 임의로 변경할 수 없게 됨

### ✅ 자식 컴포넌트에 ref 전달하기

- <button />과 같은 기본 HTML 요소가 아닌 리액트 컴포넌트에 ref를 전달할 수도 있으나, ref를 일반적인 props를 넘겨주는 방식으로 전달하면 브라우저에서 경고 메세지를 띄움
    
    → ref라는 속성의 이름은 리액트에서 DOM 요소 접근 이라는 특수한 목적으로 사용되므로 props를 넘겨주는 방식으로 전달할 수 없음 
    
- **리액트 컴포넌트에서 ref를 prop으로 전달하기 위해서는 forwardRef를 사용해야함**
    
    <aside>
    💡
    
    ref가 아닌 inputRef 등의 다른 이름을 사용한다면 forwardRef를 사용하지 않아도 됨
    
    </aside>
    
- 예시 코드
    
    ```tsx
    // ref라는 이름으로 props를 넘겨주는 코드
    import { useRef } from "react";
    
    const Component = () => {
      const ref = useRef<HTMLInputElement>(null);
      return <MyInput ref= {ref} />;
    };
    
    interface Props {
      ref: RefObject<HTMLInputElement>;
    }
    
    /**
      * 🚨 Warning: MyInput: `ref` is not a prop. Trying to access it will result in
      `undefined` being returned
      * If you need to access the same value within the child component, you should pass
      it as a different prop
      */
    const MyInput = ({ ref }: Props) => {
      return <input ref= {ref} />;
    };
    ```
    
    ```tsx
    // forwardRef를 사용한 코드
    interface Props {
      name: string;
    }
    
    const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
      return (
        <div>
          <label>{props.name}</label>
          <input ref= {ref} />
        </div>
      );
    });
    ```
    
    - 위처럼 forwardRef의 두 번째 인자에 ref를 넣어 자식 컴포넌트로 ref를 전달할 수 있음
- forwardRef의 타입 정의
    
    ```tsx
    function forwardRef<T, P = {}>(
        render: ForwardRefRenderFunction<T, PropsWithoutRef<P>>,
    ): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;
    ```
    
- forwardRef에 인자로 넘겨주는 콜백함수인 ForwardRefRenderFunction의 타입 정의
    
    ```tsx
    interface ForwardRefRenderFunction<T, P = {}> {
    	(props: P, ref: ForwardedRef<T>): ReactNode;
      displayName?: string | undefined;
      defaultProps?: never | undefined;
      propTypes?: never | undefined;
    }
    ```
    
    - ForwardRefRenderFunction은 두 개의 타입 매개변수 T와 P를 받음
    - P : 일반적인 리액트 컴포너트에서 자식 컴포넌트로 넘겨주는 props타입
    - T : ref로 전달하려는 요소의 타입
    - ref의 타입은 T를 래핑한 형태인 `ForwardedRef<T>` 타입임
        - ForwardedRef의 타입 정의
            
            ```tsx
            type ForwardedRef<T> = 
            	| ((instance: T | null) => void) 
            	| MutableRefObject<T | null> 
            	| null;
            ```
            
            - `MutableRefObject<T | null>`
                - useRef 반환 타입은 MutableRefObject<T> 또는 RefObject<T>가 될 수 있지만 **ForwardedRef에는 MutableRefObject만 들어올 수 있음**
                    
                    → MutableRefObject가 RefObject보다 넓은 범위의 타입을 가지기 때문에 부모 컴포넌트에서 ref를 어떻게 선언했는지와 관계없이 자식 컴포넌트가 해당 ref를 수용할 수 있음
                    

### ✅ useImperativeHandle

- **`ForwardRefRenderFunction`과 함께 쓸 수 있는 훅**
- 이 훅을 활용하면 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징된 메서드를 호출할 수 있게 됨
→ 자식 컴포넌트는 내부 상태나 로직을 관리하면서 부모 컴포넌트와의 결합도를 낮출 수 있음
- 예시 코드
    
    ```tsx
    // <form> 태그의 submit 함수만 따로 뽑아와서 정의
    type CreateFormHandle = Pick<HTMLFormElement, "submit">;
    type CreateFormProps = {
      defaultValues?: CreateFormValue;
    };
    
    const JobCreateForm: React.ForwardRefRenderFunction<CreateFormHandle,
      CreateFormProps> = (props, ref) => {
      // useImperativeHandle Hook을 사용해서 submit 함수를 커스터마이징
      useImperativeHandle(ref, () => ({
        submit: () => {
          /* submit 작업을 진행 */
        }
      }));  
      // ...
    }
    ```
    
    - 자식 컴포넌트에서는 ref와 정의된 CreateFormHandle을 통해 부모 컴포넌트에서 호출할 수 있는 함수를 생성함
- 부모컴포넌트에서 자식 컴포넌트의 특정 메서드를 실행하는 예시 코드
    
    ```tsx
    const CreatePage: React.FC = () => {
      // `CreateFormHandle` 형태를 가진 자식의 ref를 불러옴
      const refForm = useRef<CreateFormHandle>(null);
      
      const handleSubmitButtonClick = () => {
        // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근 가능
        refForm.current?.submit();
      };
      // ...
    };
    ```
    
    - 부모 컴포넌트에서는 다음처럼 `current.submit()` 을 사용해서 자식 컴포넌트의 특정 메서드를 실행할 수 있게됨

### ✅ useRef의 여러 가지 특성

- useRef는 자식 컴포넌트를 저장하는 변수로 활용하는 것 뿐만 다른 방식으로도 유용하게 사용 가능
    - useRef는 관리되는 변수의 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않음. 이 특성을 활용하면 상태가 변경되더라도 불필요한 리렌더링을 피할 수 있음
    - 리액트 컴포넌트의 상태는 상태 변경 함수를 호출하고 렌더링된 이후에 업데이트된 상태의 조회가 가능하지만 useRef로 관리되는 변수는 값을 설정한 후 즉시 조회 가능
- 예시코드
    
    ```tsx
    type BannerProps = {
      autoplay: boolean;
    };
    const Banner: React.FC<BannerProps> = ({ autoplay }) => {
      const isAutoPlayPause = useRef(false);  
      if (autoplay) {
        // keepAutoPlay 같이 isAutoPlay가 변하자마자 사용해야 할 때 사용 가능
        const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current;
        // ...
      }
      return (
        <>
          {autoplay && (
            <button
              aria-label="자동 재생 일시 정지"
              // isAutoPlayPause는 사실 렌더링에는 영향을 미치지 않고 로직에만 영향을 
              // 주므로 상태로 사용해서 불필요한 렌더링을 유발할 필요가 없음
              onClick={() => { isAutoPlayPause.current = true }}
            />
          )}
        </>
      );
    };
    const Label: React.FC<LabelProps> = ({ value }) => {
      useEffect(() => {
        // value.name과 value.id를 사용해서 작업
      }, [value]);
      // ...
    };
    ```
    
    - isAutoPlayPause
        - 현재 자동 재생이 일시 정지 되었는지 확인하는 ref
        - 렌더링에 영향을 미치지 않고 값이 변경되더라도 다시 렌더링할 필요없이 사용할 수 있어야함
- 훅의 규칙(리액트 훅을 안전하게 사용하기 위한 두 가지 규칙)
    - 훅은 항상 최상위 레벨에서 호출되어야함
    → 조건문, 반복문, 중첩 함수, 클래스 등 내부에서는 훅을 호출하지 않아야함
    - 훅은 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야함
- 리액트에서 훅은 호출 순서에 의존하기 때문에 모든 컴포넌트 렌더링에서 훅의 순서가 항상 동일하게 유지되어야하며 이를 통해 항상 동일한 컴포넌트 렌더링이 보장됨

## 9.2 커스텀 훅

### 1. 나만의 훅 만들기

- 리액트에서 기본적으로 제공되는 useState, useRef 등의 훅에 더해 사용자 정의 훅을 생성하여 컴포넌트 로직을 함수로 뽑아내 재사용할 수 있음
- 커스텀 훅은 리액트 컴포넌트 내에서만 사용할 수 있으며 이름은 반드시 use로 시작도ㅓㅣ어야함
- 예시 - useInput
    - 인자로 받은 초깃값을 useState로 관리하며 해당 값을 수정할 수 있는 onChange 함수를 input의 값과 함께 반환하는 훅
    
    ```jsx
    // useInput 생성 예시
    // useInput.js
    import { useState } from "react";
    const useInput = (initialValue) => {
      const [value, setValue] = useState(initialValue);
      const onChange = (e) => {
        setValue(e.target.value);
      };
      return { value, onChange };
    };
    ```
    
    ```jsx
    // useInput 사용 예시
    // App.jsx
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
    

### 2. 타입스크립트로 커스텀 훅 강화하기

- 위 useInput 예시를 타입스크립트로 작성하기
    - 기존 코드를 .ts 파일에 작성하면 아래와 같이 에러가 발생함
    
    ```tsx
    // useInput.ts
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
    
    - 타입을 명시해주면 아래와 같아짐
    
    ```tsx
    // useInput.ts
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
    
- react 타입 정의 :  https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts
