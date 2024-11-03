# 8장 - JSX에서 TSX로

## 8.1 리액트 컴포넌트의 타입

### 1. 클래스 컴포넌트 타입

- 클래스 컴포넌트가 상속받는 React.Component와 React.PureComponent의 타입 정의
    
    ```tsx
    interface Component<P = {}, S = {}, SS = any> extends ComponentLifecycle<P, S, SS> {}
    class Component<P, S> {
    	// ...
    }
    class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
    // P: propss, S : state
    ```
    
    - props와 상태 타입을 제네릭으로 받고 있음

### 2. 함수 컴포넌트 타입

- 함수 컴포넌트를 만드는 다양한 방식
    
    ```tsx
    // 함수 선언을 사용한 방식 
    function Welcome(props:WelcomeProps): JSX.Element {}
    // 함수 표현식을 사용한 방식 - React.FC 사용
    const Welcome: React.FC<WelcomeProps> = ({name}) => {};
    // 함수 표현식을 사용한 방식 - React.VFC사용
    const Welcome: React.VFC<WelcomeProps> = ({name}) => {};
    // 함수 표현식을 사용한 방식 - JSX.Element를 반환 타입으로 지정
    const Welcome = ({name}: WelcomeProps): JSX.Element => {};
    ```
    
- 함수 표현식을 사용해서 함수 컴포넌트를 선언할 때 `React.FC` 또는 `React.VFC`로 타입을 지정해서 많이 사용했음
- React.FC
    - 암묵적으로 children을 포함하고 있기 때문에 해당 컴포넌트에서 children을 사용하지 않더라도 children props를 허용
- React.VFC
    - React.FC 이후에 등장(@types/react 16.9.4 이후)
    - children props를 허용하지 않음
- **리액트 v18로 넘어오면서 React.VFC는 삭제되고 React.FC에서 children이 삭제됨**
    
    **→ React.FC 또는 props 타입, 반환 타입을 직접 지정하는 형태로 타이핑 해줘야함** 
    

### 3. Children props 타입 지정

```tsx
type PropsWithChildren<P> = P & {children?: ReactNode | undefined};
```

- 가장 보편적인 children 타입은 `ReactNode | undefined`
- `ReactNode`
    - ReactElement 외에도boolean, number 등 여러 타입을 포함하고 있는 타입
    → 구체적으로 타이핑하는 용도에는 적합하지 않음
- 구체적으로 타이핑을 하고 싶다면 children에 대해 추가로 타이핑 필요

### 4. render 메서드와 함수 컴포넌트의 반환 타입 
    - React.ReactElement vs JSX.Element vs React.ReactNode

- React.ReactElement
    - 함수 컴포넌트의 반환 타입인 ReactElement는 아래와 같이 정의됨
        
        ```tsx
        interface ReactElement<P = any,
        	T extends string | JSXElementConstructor<any> = 
        	| string 
        	| JSXElementConstructor<any>
        > {
          type: T;
          props: P;
          key: string | null;
        }
        ```
        
    - React.createElement를 호출하는 형태의 구문으로 변환하면 반환타입은 ReactElement
    - 리액트의 가상 DOM의 엘리먼트는 ReactElement 형태로 저장됨
        
        → ReactElement 타입은 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷
        
- JSX.Element
    
    ```tsx
    declare global {
    	namespace JSX {
    		interface Element extends React.ReactElement<any, any> {}
    	}
    }
    ```
    
    - JSX.Element 타입은 React.ReactElement를 확장하고 있는 타입
    - 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성을 제공
    → 컴포넌트 타입을 재정의하거나 변경하는 것이 용이해짐
- React.ReactNode
    
    ```tsx
    type ReactNode =
    	| ReactElement
    	| string
    	| number
    	| Iterable<ReactNode>
    	| ReactPortal
    	| boolean
    	| null
    	| undefined
    	| DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES[
    	    keyof DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES
    	];
    ```
    
    - ReactElement 외 string, number, boolean 등 여러 타입을 포함
    - 클래스형 컴포넌트는 render 메서드에서 ReactNode를 리턴
- React.ReactElement vs JSX.Element vs React.ReactNode 간의 포함 관계
    ![image](https://github.com/user-attachments/assets/d2296a52-edca-4d36-8b07-741813fde697)


### 5. ReactElement, ReactNode, JSX.Element 활용하기

- 전부 리액트의 요소를 나타내는 타입
- `@types/react`
    
    ```tsx
    // types/react/index.d.ts
    declare namespace React {
    	// ...
    	// React Element
    	interface ReactElement<P = any,
    		T extends string | JSXElementConstructor<any> = 
    		| string 
    		| JSXElementConstructor<any>
    	> {
    	  type: T;
    	  props: P;
    	  key: string | null;
    	}
    	// ReactNode 
    	type ReactNode =
    		| ReactElement
    		| string
    		| number
    		| Iterable<ReactNode>
    		| ReactPortal
    		| boolean
    		| null
    		| undefined
    		| DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES[
    		    keyof DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES
    		];
    	// ...
    	type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>;	
    }
    // JSX.Element
    declare global {
    	namespace JSX {
    		interface Element extends React.ReactElement<any, any> {
    			// ...
    		}
    		// ...
    	}
    }
    ```
    

### ✅ ReactElement

- JSX는 리액트 엘리먼트를 생성하기 위한 문법
- 트랜스파일러는 JSX 문법을 createElement 메서드 호출문으로 변환하여 아래와 같이 리액트 엘리먼트를 생성
    
    ```tsx
    const element = React.createElement(
      "h1",
      { className: "greeting" },
      "Hello, world!"
    );
    // 주의: 다음 구조는 단순화되었다
    const element = {
      type: "h1",
      props: {
        className: "greeting",
        children: "Hello, world!",
      },
    };
    declare global {
      namespace JSX {
        interface Element extends React.ReactElement<any, any> {
          // ...
        }
        // ...
      }
    }
    ```
    
- ReactElement 타입은 JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입
- 사용 예시
    - 아래의 JSX.Element 예시를 확장하여 추론관점에서 더 유용하게 활용할 수 있으며 원하는 컴포넌트의 props를 ReactElement의 제네릭으로 지정이 가능
        
        ```tsx
        interface IconProps {
          size: number;
        }
        interface Props {
          // ReactElement의 props 타입으로 IconProps 타입 지정
          icon: React.ReactElement<IconProps>;
        }
        const Item = ({ icon }: Props) => {
          // icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론됨
          const iconSize = icon.props.size;
          return <li>{icon}</li>;
        };
        ```
        

### ✅ ReactNode

- 리액트의 render 함수가 반환할 수 있는 모든 형태를 담고 있음
- 사용 예시
    - JSX 형태의 문법을 string, number, null, undefined 같이 어떤 타입이든 children prop으로 지정할 수 있게 하고 싶다면 ReactNode 타입으로 children을 선언하면 됨
        
        ```tsx
        type PropsWithChildren<P = unknwon> = P & {
        	children?: ReactNode | undefined;
        };
        interface MyProps {
        	// ...
        }
        type MyComponentProps = PropsWithChildren<MyProps>;
        ```
        

### ✅ JSX.Element

- ReactElement의 제네릭으로 props와 타입 필드에 대해 any 타입을 가지도록 확장
→ ReactElement의 특정 타입으로 props와 타입 필드를 any 가지는 타입
- 사용 예시
    - 리액트 엘리먼트를 prop으로 전달받아 render props 패턴으로 컴포넌트를 구현할 때 활용 가능
        
        ```tsx
        interface Props {
          icon: JSX.Element;
        }
        const Item = ({ icon }: Props) => {
          // prop으로 받은 컴포넌트의 props에 접근 가능
          const iconSize = icon.props.size;
        
          return <li>{icon}</li>;
        };
        // icon prop에는 JSX.Element 타입을 가진 요소만 할당 가능
        const App = () => {
          return <Item icon={<Icon size={14} />} />;
        };
        ```
        

### 7. 리액트에서 기본 HTML 요소 타입 활용하기

### ✅ DetailedHTMLProps와 ComponentPropsWithoutRef

- HTMl 태그의 속성 타입을 활용하는 대표적인 방법들
- DetailedHTMLProps
    
    ```tsx
    type NativeButtonProps = React.DetailedHTMLProps<
      React.ButtonHTMLAttributes<HTMLButtonElement>,
      HTMLButtonElement
    >;
    type ButtonProps = {
      onClick?: NativeButtonProps["onClick"];
    };
    ```
    
    - HTMl 태그 속성과 호환되는 타입을 쉽게 선언할 수 있음
- ComponentPropsWithoutRef
    
    ```tsx
    type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
    type ButtonProps = {
      onClick?: NativeButtonType["onClick"];
    };
    ```
    
    - 함수 컴포넌트의 props로 DetailedHTMLProps와 같이 ref를 포함하는 타입을 사용하게 되면 실제로는 동작하지 않는 ref를 받도록 타입이 지정되어 예기치 않은 에러가 발생할 수 있음
    → HTML 속성을 확장하는 props를 설계할 땐 ComponentPropsWithoutRef 타입을 사용해서 ref가 실제로 forwardRef와 함께 사용될 때만 props로 전달되도록 타입을 정의하는 것이 안전함

## 8.2 타입 스크립트로 리액트 컴포넌트 만들기

- 타입스크립트는 리액트 프로젝트에서 공통 컴포넌트에 어떤 타입의 속성이 제공되어야하는지 알려주며 필수로 전달되어야 하는 속성이 전달되지 않았을 때 에러를 표시하여 실수를 예방하도록 도움을 줌

### 1. JSX로 구현된 Select 컴포넌트

```tsx
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

- 각 option의 key, value 쌍을 객체로 받고 있음
- 선택된 값이 변경될 때 호출되는 onChange 이벤트 핸들러를 받도록 구현되어있음
- **추가적인 설명이 없다면 컴포넌트를 사용하는 입장에서 각 속성에 어떤 타입의 값을 전달해야하는지 알기 어렵다는 문제가 존재**

### 2. JSDocs로 일부 타입 지정하기

- 컴포넌트의 속성 타입을 명시하기 위해 JSDocs를 사용할 수 있음
    
    ```tsx
    /**
    * Select 컴포넌트
    * @param {Object} props - Select 컴포넌트로 넘겨주는 속성
    * @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
    * @param {string | undefined} props.selectedOption - 현재 선택된 option의 key값 (optional)
    * @param {function} props.onChange - select 값이 변경되었을 때 불리는 callBack 함수 (optional)
    * @returns {JSX.Element} 
    */
    const Select = //...
    ```
    
    - 하지만 options가 어떤 형식의 객체를 나타내는지, onChange의 매개변수 및 반환 값에 대한 구체적인 정보를 알기 어려움

### 3. props 인터페이스 적용하기

- JSX 파일의 확장자를 TSX로 변경 후 Select 컴포넌트의 props에 대한 인터페이스 작성
    
    ```tsx
    type Option = Record<string, string>; // {[key: string]: string}
    interface SelectProps {
      options: Option;
      selectedOption?: string;
      onChange?: (selected?: string) => void;
    }
    const Select = ({
      options,
      selectedOption,
      onChange,
    }: SelectProps): JSX.Element => {
      //...
    };
    ```
    
    - Record는 key, value의 타입 모두 string인 객체 타입을 생성하는 유틸리티 타입으로 사용됨
    - options의 타입을 정의해줌으로써 string이 아닌 배열이나 다른 유형의 value를 가진 객체는 전달할 수 없음
    - onChange는 선택된 string 값(또는 undefined)을 매개변수로 받고 어떤 값도 반환하지 않는 함수임을 명확하게 표현
    - 또한 onChange는 옵셔널 프로퍼티이기 때문에 부모컴포넌트에서 넘겨주지않아도 해당 컴포넌트를 사용할 수 있음

### 4. 리액트 이벤트

- 리액트는 가상 DOM을 다루면서 이벤트도 별도로 관리함
- 리액트 컴포넌트(노드)에 등록되는 이벤트 리스너는 onClick, onChange처럼 카멜 케이스로 표기
→ 리액트 이벤트는 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지는 않음
- 리액트는 브라우저 이벤트를 합성한 합성 이벤트(SyntheticEvent)를 제공함
- 리액트 컴포넌트에 연결할 이벤트 핸들러도 해당 타입을 일치시켜줘야함
    
    ```tsx
    const Select = ({ onChange, options, selectedOption }: SelectProps) => {
      const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
      // ChangeEventHandler<HTMLSelectElement> 타입의 이벤트를 
      // 매개변수로 받아서 해당 이벤트를 처리하는 핸들러 작성 가능
        const selected = Object.entries(options).find(
          ([_, value]) => value === e.target.value
        )?.[0];
        onChange?.(selected);
      };
    
      return <select onChange={handleChange}>{/* ... */}</select>;
    };
    ```
    
    - `React.ChangeEventHandler<HTMLSelectElement>` 타입은 `React.EventHandler<ChangeEvent<HTMLSelectElement>>`와 동일한 타입
    - onChange는 HTML의 select 엘리먼트에서 발생하는 change 이벤트에 대한 핸들러로 선언됨

### 5. 훅에 타입 추가하기

- useState 같은 함수도 타입 매개변수를 지정해줌으로써 반환되는 state 타입 지정이 가능
- 타입 매개변수로 좀 더 명확한 타입을 지정함으로써 다른 개발자가 해당 state나 changeState를 한정된 타입으로만 다룰 수 있도록 강제할 수 있음
- 타입을 지정하지 않은 useState
    
    ```tsx
    const [fruit, changeFruit] = useState("apple");
    
    // error가 아님
    const func = () => {
      changeFruit("orange");
    };
    ```
    
- 타입을 지정한 useState
    
    ```tsx
    const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };
    type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
    const [fruit, changeFruit] = useState<Fruit | undefined>("apple");
    
    // 에러 발생
    const func = () => {
      changeFruit("orange");
    };
    ```
    
    - keyof typeof fruits를 사용하여 fruits 키값만 추출해서 Fruit라는 타입을 새로 만든 후 useState의 제네릭으로 활용함
    → changeFruit에 'apple' , 'banana' , 'blueberry', undefined를 제외한 다른 값이 할당되면 에러가 발생하도록 설정

### 6. 제네릭 컴포넌트 만들기

```tsx
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption="orange" />
  );
};
```

- 위 코드에서 selectedOption은 options에 존재하지 않는 값을 받아도 오류가 발생하지 않음
- 하지만 changeFruit의 매개변수 Fruit는 prop으로 전달돼야 하는 onChange의 매개변수 타입인 string보다 좁기 때문에 타입 에러가 발생함
- Select를 사용하는 입장에서 제한된 key, value만을 가지도록 하기 위해 제네릭을 사용하여 컴포넌트를 만들 수 있음
- 객체 형식의 타입을 받아 해당 객체의 키값은 selectedOption, onChange의 매개변수에만 적용하도록 만든 예시
    
    ```tsx
    interface SelectProps<OptionType extends Record<string, string>> {
      options: OptionType;
      selectedOption?: keyof OptionType;
      onChange?: (selected?: keyof OptionType) => void;
    }
    
    const Select = <OptionType extends Record<string, string>>({
      options,
      selectedOption,
      onChange,
    }: SelectProps<OptionType>) => {
      // Select component implementation
    };
    ```
    
    - Select 컴포넌트에 전달되는 props의 타입 기반으로 타입이 추론되어 `<Select<추론된_타입>>` 형태의 컴포넌트가 생성됨

### 7. HTMLAttributes, ReactProps 적용하기

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;
// ComponentPropsWithoutRef는 리액트 컴포넌트의 props 타입을 반환해주는 타입
interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  // ...
}
```

- Type[’key’]를 활용하면 객체 형식의 타입 내부 속성값을 가져올 수 있음

### 8. styled-components를 활용한 스타일 정의

- 리액트 컴포넌트를 만들 때 CSS 파일 대신 자바스크립트 안에 직접 스타일을 정의하는 CSS-in-JS 기법을 사용할 수 있음.
- styled-components는 대표적인 CSS-in-JS 라이브러리

### 9. 공변성과 반공변성

- 일반적인 타입들은 공변성을 가지고 있어 좁은 타입에서 넓은 타입으로의 확장이 가능함
- 제네릭 타입을 지닌 함수는 반공변성을 가짐
→ T<B>가 T<A>의 서브타입이 되어 좁은 타입 T<A>의 함수를 넓은 타입 T<B>의 함수에 적용할 수 없음
- 안전한 타입 가드를 위해 일반적으로 반공변적인 함수 타입을 설정하는 것이 권장됨

## 8.3 정리

- 최종적으로 완성된 Select 컴포넌트의 모습
  
  ```tsx
  import React, { useState } from "react";
  import styled from "styled-components";
  
  const theme = {
    fontSize: {
      default: "16px",
      small: "14px",
      large: "18px",
    },
    color: {
      white: "#FFFFFF",
      black: "#000000",
    },
  };
  
  type Theme = typeof theme;
  
  type FontSize = keyof Theme["fontSize"];
  type Color = keyof Theme["color"];
  
  interface SelectStyleProps {
    color: Color;
    fontSize: FontSize;
  }
  
  const StyledSelect = styled.select<SelectStyleProps>`
    color: ${({ color }) => theme.color[color]};
    font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
  `;
  
  type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;
  
  interface SelectProps<OptionType extends Record<string, string>>
    extends Partial<SelectStyleProps> {
    id?: ReactSelectProps["id"];
    className?: ReactSelectProps["className"];
    options: OptionType;
    selectedOption?: keyof OptionType;
    onChange?: (selected?: keyof OptionType) => void;
  }
  
  const Select = <OptionType extends Record<string, string>>({
    className,
    id,
    options,
    onChange,
    selectedOption,
    fontSize = "default",
    color = "black",
  }: SelectProps<OptionType>) => {
    const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
      const selected = Object.entries(options).find(
        ([_, value]) => value === e.target.value
      )?.[0];
      onChange?.(selected);
    };
  
    return (
      <StyledSelect
        id={id}
        className={className}
        fontSize={fontSize}
        color={color}
        onChange={handleChange}
        value={selectedOption && options[selectedOption]}
      >
        {Object.entries(options).map(([key, value]) => (
          <option key={key} value={value}>
            {value}
          </option>
        ))}
      </StyledSelect>
    );
  };
  
  const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };
  
  type Fruit = keyof typeof fruits;
  
  const FruitSelect = () => {
    const [fruit, changeFruit] = useState<Fruit | undefined>();
  
    return (
      <Select
        className="fruitSelectBox"
        options={fruits}
        onChange={changeFruit}
        selectedOption={fruit}
        fontSize="large"
      />
    );
  };
  
  export default FruitSelect;
  
  ```
