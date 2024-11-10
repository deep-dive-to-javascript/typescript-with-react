# 훅

## 리액트 훅

리액트 휵이 도입되며 함수 컴포넌트에서도 컴포넌트의 생명주기에 맞춰 로직 실행이 가능하게 되었다. 

### useState

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

배열 요소에는 없는 agee 라는 잘못된 속성이 포함된 객체로 인해 예상치 못한 사이드 이펙트가 발생할 수 있음.

⇒ 타입 스크립트를 사용하여 해당 에러 방지 가능

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

### 의존성 배열을 사용하는 훅 : useEffect, useLayoutEffect

useEffect : 렌더링 후 함수 컴포넌트 조건부로 실행시키고 싶을 때 사용

```tsx
function useEffect(effect: EffectCallback, deps?: DependencyList): void;
type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

- EffectCallback : Destructor 반환 또는 아무것도 반환하지 않는 함수
- useEffect 콜백 함수에는 비동기 함수가 들어갈 수 없음 ⇒ 경쟁 상태(여러 프로세스가 공유된 자원에 접근하며 실행 순서를 예측할 수 없게 되는 문제) 예방 가능
- deps : effect가 수행되기 위한 조건 ⇒ 얕은 비교를 통하기 때문에 기본 자료형이 아닌 객체, 배열일 때는 참조값이 아닌 실제로 사용되는 프로퍼티 값을 사용해야함.

```tsx
type SomeObject = {
  name: string;
  id: string;
};

interface LabelProps {
  value: SomeObject;
}

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
   // value 객체를 의존성 배열에 넣는 것은 비추천
    // 이유: 객체 참조값 비교로 인해 내부 속성 변경을 감지하지 못할 수 있음
  }, [value]);
  
  // ...
};

const { id, name } = value;

useEffect(() => {
  // value.name과 value.id 대신 name, id를 직접 사용한다
}, [id, name]);
```

useLayoutEffect : 레이아웃 배치와 화면 렌더링이 모두 완료된 후 실행

```tsx
type DependencyList = ReadonlyArray<any>;

function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;
```

```tsx
const [name, setName] = useState("");

useEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래의 setName()을 실행한다고 생각하자
  setName("배달이");
}, []);

return (
  <div>
    {안녕하세요, ${name}님!}
  </div>
);

사용자에게 보이는 화면 변화 : 안녕하세요,  님! => 안녕하세요, 배달이님!
```

```tsx
import React, { useState, useLayoutEffect } from "react";

const App = () => {
  const [name, setName] = useState("");

  useLayoutEffect(() => {
  
    setName("배달이");
  }, []);

  return (
    <div>
      {`안녕하세요, ${name}님!`}
    </div>
  );
};

export default App;

사용자가 보는 화면 : 안녕하세요, 배달이님!

```

| **단계** | **`useEffect`** | **`useLayoutEffect`** |
| --- | --- | --- |
| DOM 렌더링 전 | Virtual DOM 생성 | Virtual DOM 생성 |
| 초기 상태 브라우저 반영 | **초기 상태 반영 후 `useEffect` 실행** | **초기 상태 반영 전에 실행** |
| DOM 업데이트 및 재렌더링 | 상태 변경 후 DOM 재렌더링 및 반영 | 상태 변경 후 DOM 재렌더링 및 반영 |

### useMemo, useCallback : 이전에 생성된 값 또는 함수를 기억하며 동일한 값의 함수의 반복 생성을 방지하는 훅

```tsx
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;
```

- useEffect와 마찬가지로 deps 가 객체 또는 배열일 때, 얕은 비교임을 주의하여 불필요한 변경이 발생하지 않도록 해야함.

### useRef : 요소에 포커스 설정, 특정 컴포넌트의 위치로 스크롤하는 등 DOM을 직접 선택해야하는 경우 사용하는 훅

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

초기 설정에 null값을 넣을 수 있는 이유를 useRef 타입 정의에서 확인 가능 ⇒ initialValue: T | null (초기 렌더링 시점에는 참조할 DOM 요소가 없기 때문에, 타입 정의에서 `null`을 허용)

```tsx
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}

interface RefObject<T> {
  readonly current: T | null; // => ref.current값 임의 변경 가능
}
```

`useRef`의 첫 번째 시그니처와 두 번째 시그니처는 각각 다른 사용 사례를 지원하기 위해 존재

- 첫 번째 시그니처: 초기값이 반드시 필요한 상황 (렌더링과 무관한 값 저장).
    
    ```tsx
    const MyComponent = () => {
      const countRef = useRef(0); // 초기값을 0으로 설정
    
      const handleClick = () => {
        countRef.current += 1; // 값 업데이트 (DOM 요소 참조 X, 재렌더링과 무관)
        console.log(`Clicked ${countRef.current} times`);
      };
    
      return <button onClick={handleClick}>Click Me</button>;
    };
    
    ```
    
- 두 번째 시그니처: 초기값이 없거나 `null`이어야 하는 상황 (DOM 요소 참조).

자식 컴포넌트에 ref 전달하기 : 리액트 컴포넌트에 ref 전달

⇒ props 넘겨주는 방식 아닌 forwardRef 사용하기

```tsx
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

인자로 넘겨주는 콜백함수 ForwardRefRenderFunction 타입 정의

```tsx
interface ForwardRefRenderFunction<T, P = {}> {
  (props: P, ref: ForwardedRef<T>): ReactElement | null;
  displayName?: string | undefined;
  defaultProps?: never | undefined;
  propTypes?: never | undefined;
}
```

ref 타입 ForwardedRef의 타입 정의 : MutableRefObject로 RefObect(readonly)보다 더 넓은 타입을 가지므로 부모 컴포넌트의 ref 타입에 관계없이 자식 컴포넌트는 해당 ref 수용 가능

```tsx
type ForwardedRef<T> =
| ((instance: T | null) => void)
| MutableRefObject<T | null>
| null;
```

useImperativeHandle : 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징한 메서드를 호출 가능

| **특성** | **`ForwardRefRenderFunction`** | **`useImperativeHandle`** |
| --- | --- | --- |
| **목적** | 자식이 부모로부터 전달받은 `ref`를 사용할 때 | 부모가 자식 컴포넌트의 속성/메서드를 제어 |
| **`ref`의 방향** | **부모 → 자식** | **자식 → 부모** |
| **기본 사용 사례** | 부모가 자식의 DOM 요소를 제어 | 부모가 자식의 커스텀 메서드나 속성을 호출 |
| **주요 훅/함수** | `React.forwardRef` | `React.forwardRef` + `useImperativeHandle` |
| **동작 제어 대상** | DOM 요소 | DOM 요소 + 커스텀 메서드 |

## 커스텀 훅

### 나만의 훅 만들기 : 기본적으로 제공되는 훅에 목적을 더해 사용자 정의 훅을 생성하여 컴포넌트 로직을 함수로 뽑아내 재사용 가능

### 타입스크립트로 커스텀 훅 강화 가능

```tsx
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
