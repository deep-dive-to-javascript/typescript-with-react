# 8장 JSX에서 TSX로

2024.11.03 dasom



## 8.1 리액트 컴포넌트의 타입

@types/react 패키지에 정의된 리액트 내장 타입 중, 헷갈릴 수 있는 리액트 컴포넌트 타입을 살펴보자.

### 1. 클래스 컴포넌트 타입

```ts
// 타입 정의
interface Component<P = {}, S = {}, SS = any> extends ComponentLifecycle<P, S, SS>{}

class Component<P,S> {
  /* ...생략 */
}

class PureComponent<P ={}, S= {}, SS=any> extends Components<P, S, SS>{}

/*
* P: props, S: sate
* SS: Snapshot State(컴포넌트 업데이트 직전 DOM 상태를 스냅샷으로 저장할 수 있게 해주며, 이를 이후 로직에서 활용할 수 있다.)
*/
```



> 🤔 그럼 ComponentLifecycle의 타입은 어떻게 생겼을까?

```ts
    interface ComponentLifecycle<P, S, SS = any> extends NewLifecycle<P, S, SS>, DeprecatedLifecycle<P, S> {
        /**
         * Called immediately after a component is mounted. Setting state here will trigger re-rendering.
         */
        componentDidMount?(): void;
        /**
         * Called to determine whether the change in props and state should trigger a re-render.
         *
         * `Component` always returns true.
         * `PureComponent` implements a shallow comparison on props and state and returns true if any
         * props or states have changed.
         *
         * If false is returned, `Component#render`, `componentWillUpdate`
         * and `componentDidUpdate` will not be called.
         */
        shouldComponentUpdate?(nextProps: Readonly<P>, nextState: Readonly<S>, nextContext: any): boolean;
        /**
         * Called immediately before a component is destroyed. Perform any necessary cleanup in this method, such as
         * cancelled network requests, or cleaning up any DOM elements created in `componentDidMount`.
         */
        componentWillUnmount?(): void;
        /**
         * Catches exceptions generated in descendant components. Unhandled exceptions will cause
         * the entire component tree to unmount.
         */
        componentDidCatch?(error: Error, errorInfo: ErrorInfo): void;
    }
```



> 🤔 SnapShot 더 알아보기!

```ts
class StringSnapshotComponent extends Component<{}, {}, string> {
  getSnapshotBeforeUpdate(prevProps: {}, prevState: {}): string {
    return "snapshot 상태를 문자열로 저장";
  }

  componentDidUpdate(prevProps: {}, prevState: {}, snapshot: string) {
    console.log("Snapshot before update:", snapshot); // 업데이트 전의 스냅샷 위치를 반환함. 이를 통해 원래 위치로 스크롤을 이동하는 등의 로직 처리가 가능해짐
  }

  render() {
    return <div>문자열 스냅샷 예제 by ChatGPT</div>;
  }
}

```



> 🤔 그래서 Component와 PureComponent의 차이는?
>
> `Component`: (상태 또는 props 변경에 따른) 모든 업데이트에 대해 렌더링이 수행된다. 필요한 경우에만 렌더링 해 최적화가 필요하다면 `shouldComponentUpdate` 메서드를 수동으로 구현해야 한다.
>
> `PureComponent`: `shouldComponentUpdate`가 이미 구현되어 있어 얕은 비교(shallow comparison)를 통해 불필요한 렌더링을 방지한다. 하지만 깊은 비교가 가능하지 않아 복잡한 데이터 구조의 경우 side effect가 발생할 수 있다.



### 2. 함수 컴포넌트 타입

```ts
// 함수 선언
function Welcome(props: WelcomProps): JSX.Element{}

// 함수 표현식 - React.FC
const Welcom: React.FC<WelcomeProps> = ({name}) => {};

// 함수 표현식 - React.VFC
const Welcom: React.VFC<WelcomeProps> = ({name}) => {};

// 함수 표현식 - JSX.Element
const Welcome = ({name}: WelcomProps): JSX.Element => {}
```

```ts
// React.FC
type FC<P={}> = FunctionComponent<P>
interface FunctionComponent<P={}>{
  // props에 children을 추가
  (props: PropsWithCildren<P>, context? any): ReactElement<any, any> | null;
  // 후략..                    
}

// React.VFC
type VFC<P={}> = VoidFunctionComponent<P>
interface VoidFunctionComponent<P={}>{
  // children 없음
  (props: P, context? any): ReactElement<any, any> | null;
  // 후략..                    
}

/*
* 기존에는 children이 없는 컴포넌트에 React.VFC 를 사용하는 방식으로 사용했다.
* 하지만 React 18 이후부터 React.VFC가 사라지고, React.FC에서 children이 사라졌다. 따라서 React.FC에 pops 타입과 반환타입을 지정하는 형태로 타이핑 해서 사용해야 한다.
*/
```



> 🤔 변경 후의 사용법을 알고 싶은데요?

```tsx
import React from 'react';

// Props 타입 정의
interface MyComponentProps {
  title: string;
  onClick: () => void;
  children: React.ReactNode; // children 타입을 명시적으로 추가
}

// React.FC를 사용해 props와 반환 타입을 지정
const MyComponent: React.FC<MyComponentProps> = ({ title, onClick, children }) => {
  return (
    <div>
      <h1>{title}</h1>
      <button onClick={onClick}>Click me</button>
      <div>{children}</div> {/* children을 렌더링 */}
    </div>
  );
};

export default MyComponent;
```



### 3. Children props 타입 지정

가장 보편적인 children 타입은 ReactNode | undefined 이다. ReactNode는 ReactElement외에도 boolean, number 등 여러 타입을 포함하고 있어 더 구체적으로 타이핑 하고 싶다면 children에 사용할 타입을 지정하면 된다.

```ts
interface MyComponentProps {
  children: React.ReactNode;
}

interface MyComponentProps {
  children: "다솜" | "결" | "현경" | "해리" | "석영" | "여진";
}
```



### 4. render 메서드와 함수 컴포넌트의 반환 타입  - React.ReactElement vs JSX.Element vs React.ReactNode

ReactElement는 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷이다.

(가상 DOM의 엘리먼트는 ReactElement의 형태로 저장된다.)

```ts
interface ReactElement<P = any
	T extends string | JSXElementConstructor<any> =
 | string
 | JSXElementConstructor<any>
> {
	type: T;
 	props: P;
 	key: Key | null;
}
```



JSX.Element 타입은 ReactElement를 확장하고 있는 타입이며, 글로벌 네임스페이스에 정의되어 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성을 제공한다

```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any>{}
  }
}
```



React.Node는 문자열, 숫자, React 요소, 배열 등 모든 렌더링 가능한 타입을 포괄하는 타입니다.

```ts
type ReactNode = 
	| ReactChild
	| ReactFragment
	| ReactPortal
	| boolean
	| null
	| undefined;
```



> 포함 관계 정리
>
> JSX.Element ⊂ ReactElement ⊂ ReactNode



### 5. ReactElement, ReactNode, JSX.Element 활용하기

**ReactElement**

JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입

> JSX?
>
> 리액트에서 UI를 표현하기 위해 사용되는 자바스크립트의 확장문법이다.
>
> XML과 유사한 구조, HTML과 유사한 문법으로 쉽고 간결한 코드 작성이 가능하다. 

**ReactNode**

cf. ReactChild 타입: ReactElement | string | number 으로 ReactElement보다 넓은 범위

ReactNode: ReactChild | ReactFragmanet... ReactChild보다 넓은 범위

**JSX.Element**

ReactElement의 특정 타입으로 props와 타입 필드를 any로 가지는 타입

=> 결국 세가지 모두 리액트에서 제공하는 컴포넌트를 나타내는 역할



### 6. 사용 예시

**ReactNode**

prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고 싶을 때 사용

```ts
interface MyComponentProps {
  children: React.ReactNode;
}
 
```

**JSX.Element**

props와 타입 필드가 any 타입인 리액트 엘리먼트를 나타내므로, 리액트 엘리먼트를 prop으로 전달받아 render props 패턴으로 컴포넌트를 구현할 때 활용

(예시 from ChatGPT)

```tsx
interface ButtonWithIconProps {
  icon: JSX.Element; // JSX.Element 타입의 icon을 prop으로 받음
  label: string;
  onClick: () => void;
}

const ButtonWithIcon: React.FC<ButtonWithIconProps> = ({ icon, label, onClick }) => {
  return (
    <button onClick={onClick} style={{ display: 'flex', alignItems: 'center', padding: '8px' }}>
      {icon} {/* 전달된 icon JSX.Element를 렌더링 */}
      <span style={{ marginLeft: '8px' }}>{label}</span>
    </button>
  );
};

// 사용 예시
const App: React.FC = () => {
  return (
    <ButtonWithIcon
      icon={<span role="img" aria-label="star">⭐</span>} // icon prop으로 JSX.Element 전달
      label="Star Button"
      onClick={() => alert('Button clicked!')}
    />
  );
};

export default App;
```

**ReactElement**

React.ReactElement는 **`type`, `props`, `key` 필드를 가진 객체**로 정의된다. 따라서 `React.ReactElement`는 더 구체적인 타입 검사를 수행한다.

-> 따라서 어떤 props가 있는지 추론되어 IDE에 표시된다

```tsx
// IconProps 타입 정의
interface IconProps {
  color?: string;
  size?: number;
}

// ButtonWithIcon의 props 타입 정의
interface ButtonWithIconProps {
  icon: ReactElement<IconProps>; // ReactElement에 IconProps 타입 지정
  label: string;
  onClick: () => void;
}

// ButtonWithIcon 컴포넌트 정의
const ButtonWithIcon: React.FC<ButtonWithIconProps> = ({ icon, label, onClick }) => {
  return (
    <button onClick={onClick} style={{ display: 'flex', alignItems: 'center', padding: '8px' }}>
      {icon} {/* icon을 JSX.Element로 렌더링 */}
      <span style={{ marginLeft: '8px' }}>{label}</span>
    </button>
  );
};

// Icon 컴포넌트 정의
const Icon: React.FC<IconProps> = ({ color = 'black', size = 24 }) => (
  <span style={{ color, fontSize: size }}>⭐</span>
);

```



### 7. 리액트에서 기본 HTML 요소 타입 활용하기

**DetailedHTMLProps와 ComponentWithoutRef **- HTML 태그 속성 타입을 활용하는 방법

DetailedHTMLProps

```ts
type NativeButtonProps = REact.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
}
```

ComponentWithoutRef

```ts
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
}
```



**언제 ComponentWithoutRef를 사용하면 좋을까**

Button 컴포넌트를 만들고 있다고 가정한다.

함수형 컴포넌트의 경우, ref를 prop으로 전달하더라도 전달받은 ref가 Button컴포넌트의 button 태그를 바라보지 않는다. 함수컴포넌트에서는 생성된 인스턴스가 없기 때문이다. 따라서 React.forwardRef 메서드를 사용해야 하는데,

HTMLProps, DetailedHTMLProps, ComponentPropsWithRef 등은 ref 속성을 포함한다. 따라서 ref가 실제로 forwardRef와 함께 사용할 때만 동작할 수 있도록, ComponentWithoutRef를 사용하는 것이 안전하다.



## 8.2 타입스크립트로 리액트 컴포넌트 만들기



### 1. JSX로 구현된 Select 컴포넌트

```jsx
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange (e) => {
    const selected = Object.entries(options)
     .find(([_, value]) => value === e.target.value)?.[0]l
     onChange?.(selected)
  };
  
  return (
  <Select
    onChange={handleChange}
value={selectOption && options[selectOption]}
   >
  	{Object.entries(options).map(([key, value]) => (
    	<option key={key} value={value}>
      	{value}
      </option>
    ))}	
  </Select>
  )
}
```



### 2. JSDocs로 일부 타입 지정하기

JSDocs를 활용해 컴포넌트에 대한 설명과 각 속성의 역할에 대해 간단하게 알려줄 수 있다

```tsx
/**
* Select 컴포넌트
* @param {Object} props - Select 컴포넌트로 넘겨주는 속성
* @param {Object} props.options - { [key: stirng]: string} 형식으로 이루어진 option 객체
* ... 중략
* @return {JSX.Element}
*/
```



### 3. props 인터페이스 적용하기

```tsx
type Option = Record<string, string>; // {[key stirng]: string}

interface Props {
  options: Option;
  selectOption?: string;
  onChange?: (selected?: stirng) => void;
}

const Select = ({ onChange, options, selectedOption }: Props): JSX.Element => 
 // ...
```



### 4. 리액트 이벤트

리액트 컴포넌트(노드)에 등록되는 이벤트 리스너는 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지 않는다. 예를 들어 이벤트 핸들러는 이벤트 버블링 단계에서 호출된다. 이벤트 캡쳐 단계에서 등록하기 위해서는 onClickCapture과 같이 일반 이벤트 리스너 이름 뒤에 Capture를 붙여야 한다.

> 🤔 이벤트 버블링과 캡처링?

```jsx
<div onClick={() => console.log("Div")}>
  <button onClick={() => console.log("Button")} />
</div>
```

> 이벤트 버블링: 이벤트가 **가장 안쪽의 요소에서 시작해 바깥쪽으로 전파**되는 방식 
>
> ​	-> Button과 Div가 순서대로 출력된다
>
> 이벤트 캡처링: 이벤트가 **가장 바깥쪽 요소에서 시작해 안쪽 요소로 전파**되는 방식
>
> ​	-> Div와 Button이 순서대로 출력된다.



```tsx
const Select = ({ onChange, options, selectedOption }: Props): JSX.Element => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> (e) => {
    const selected = Object.entries(options)
     .find(([_, value]) => value === e.target.value)?.[0]l
     onChange?.(selected)
  };
  
  return (
  <Select
    onChange={handleChange}>
		{/*...*/}
  </Select>
  )
}
```



### 5. 훅에 타입 추가하기

Select 컴포넌트를 사용해 과일을 선택할 수 있는 컴포넌트를 만들어보자.

```tsx
const fruits = {
  apple: "사과",
  banana: "바나나",
  peach: "복숭아",
}

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string|undefined>()
  /*
  * useState 뒤에 type을 지정하지 않았다면 fruit의 타입이 undefined로 추론되어 오류가 발생했을 것이다. 
  해당 타입을 더 특정하고 싶다면 type을 아래와 같이 수정할 수 있다.
  type Fruit = keyof typeof fruits;
 const [fruit, changeFruit] = useState<Fruit|undefined>()
  */
  
  return (
  <Select onChange={changeFruit} optiosn={fruits} selectedOption={fruit} />
  )
}
```



### 6. 제네릭 컴포넌트 만들기

```tsx
const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<Fruit|undefined>()
  
  return (
  <Select onChange={changeFruit} optiosn={fruits} selectedOption={fruit} />
  )
}
```

위와 같이 코드를 수정한 경우, Select 컴포넌트에서 받는 onChange 매개변수 타입인 string보다 Fruit 타입이 좁아 타입 에러가 발생한다. 따라서 제한된 키와 값만을 가지게 하기 위해 제네릭을 사용해 컴포넌트를 만들 수 있다.

```tsx
interface Props<OptionType extends Record<string, string>> {
  options: OptionType;
  selectOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = ({ onChange, options, selectedOption }: Props<OptionType>): JSX.Element => 
 // ...

// 사용
const FruitSelect: VFC = () => {
  // <Select<Fruit .../> 와 같이 작성해도 되지만, 넘겨주는 props의 타입으로 타입 추론을 해준다.
return (
		<Select options={fruits} onChange={changeFruit} selectOption="apple" />
	)
}

```



### 7. HTML Attributes, ReactProps 적용하기

className, id 같은 리액트 컴포넌트의 기본 props는 직접 적어도 되지만 리액트에서 제공하는 타입을 사용하면 더 정확하게 설정할 수 있다.

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface Props<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["calssName"];
}

// 여러개의 타입을 가져올 경우 Pick을 사용하 ㄹ수도 있다.
interface Props<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, "id" | "key" | /*...*/> {
    // ...
  }
```



### 8. styled-components를 활용한 스타일 정의

CSS-in-JS(CSS 파일 대신 자바스크립트 안에 직접 스타일을 정의) 라이브러리인 styled-component를 활용해 스타일 관련 타입을 추가

```tsx
// 우선 프로젝트에 사용될 theme 객체를 생성
const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px"
  },
  color: {
    white: "#fff",
    black: "#000"
  }
}

type Theme = typeof theme;

type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];
```

```tsx
interface SelectStyleProps {
  color: Color;
  fontSize: fontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
	color: ${({color}) => theme.color[color]};
	font-size: ${({fontSize}) => theme.fontSize[fontSize]};
`
```

Partial<Type>을 사용하면 객체 형식의 타입 내 모든 속성이 옵셔널로 설정된다.

```tsx
interface Props extends Partial<SelectStyleProps>{
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize: "default",
  color: "default",
  // ...
}: Props<OptionType>) -> {
          // ...
})
```



### 9. 공변성과 반공변성

```ts
interface User {
  id: string;
}

interface Member extends User {
  nickName: string;
}

// Member는 User의 서브타입이다 (더 좁은 타입)
```

공변성(Covariance): 타입 A가 타입 B의 **서브타입**일 때, `A` 타입이 필요할 때 `B` 타입을 사용할 수 있는 성질. 함수의 **반환 타입**은 공변성을 가진다.

반공변성(Contravariance): 타입 A가 타입 B의 **서브타입**일 때, `A` 타입이 필요할 때 `B` 타입을 사용할 수 없는 성질. 함수의 **파라미터 타입**은 반공변성을 가진다.

```tsx
interface Props<T extends string>{
  onChangeA?: (selected: T) => void; // 화살표 표기법으로 작성하면 반공병성을 띔 (권장 방식), 선택적 프로퍼티(undefined를 허용하는 선택적 값)로 선언되어 있어 에러가 발생함.
  onChangeB?: (selected: T): void; // 이변성 (공변성 + 반공변성)
}

const Component = () => {
  const changeToPineApple = (selectedApple: "apple") => {
    console.log("this is pine + selectedApple)
  }
                
  return (
 		<Select
    	// Error
      // onChangeA={changeToPineApple}
      // OK
      onChangeB={changeToPineApple}
    />
 )
}
```



