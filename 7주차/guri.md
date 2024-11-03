# 1. 리액트 컴포넌트의 타입

## 1-1 클래스 컴포넌트 타입

```tsx
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
  /* ... 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

- React.Component와 React.PureComponent의 타입 정의는 위와 같으며 P와 S는 각각 props와 상태를 의미한다.
- 상태가 있는 컴포넌트일때 제네릭의 두번째 인자로 타입을 넘겨주면 상태에 대한 타입을 지정할 수 있다.

## 1-2 함수 컴포넌트 타입

- 함수 표현식에서 함수 컴포넌트를 선언할때 가장 많이 볼 수 있는 형태는 React.FC 또는 React.VFC로 타입을 지정하는 것이다.
- React.FC에서 props 타입, 변환 타입을 직접 지정하는 형태로 타이핑해줘야함.

## 1-3 Children props 타입 지정

- 가장 보편적인 children 타입은 ReactNode | undefined가 된다.
- ReactNode는 ReactElement외에도 boolean, number등 여러 타입을 포함한다.

## 1-4 render 메서드와 함수 컴포넌트의 반환 타입 - React.ReactElement vs JSX.Element vs React.ReactNode

- ReactElement

```tsx
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

- React.createElement를 호출하는 형태의 구문으로 변환하면 React.createElement의 반환 타입은 ReactElement이다.
- ReactElement 타입은 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷이다.

- JSX.Element

```tsx
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

- ReactElement를 확장하고 있는 타입.
- 글로벌 네임스페이스에 정의 : 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성 제공

- ReactNode

```tsx
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;

type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

- ReactElement외에도 boolean, string, number등 여러 타입을 포함하고 있다.

## 1-5 ReactElement, ReactNode, JSX.Element활용하기

- ReactElement
    - JSX는 리액트 엘리먼트를 생성하기 위한 문법이며 트랜스파일러는 JSX문법을 createElement 메서드 호출문으로 변환하여 아래와 같이 리액트 앨리먼트를 생성함.
    
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
    
    - 리액트는 이런식으로 만들어진 리액트 엘리먼트 객체를 읽어 DOM을 구성함.
    - ReactElement 타입은 JSX의 createElement메서드 호출로 생성된 리액트 엘리먼트를 나타대는 타입임.
- ReactNode
    - ReactChild외에도 boolean, null, undefined등 훨씬 넓은 범주의 타입을 포함함.
    - 리액트의 render 함수가 반환할 수 있는 모든 형태를 담고 있다
- JSX.Element
    - JSX.Element는 ReactElement의 제네릭으로 props와 타입 필드에 대해 any 타입을 가지도록 확장하고 있음.

## 1-6 사용 예시

- ReactNode
    
    ```tsx
    interface MyComponentProps {
      children?: React.ReactNode;
      // ...
    }
    ```
    
    - JSX형태의 문법을 어떤 타입이든 childrean prop으로 지정할수 있게 하고 싶을때 ReactNode타입으로 children을 선언하면 됨
    - prop으로 리액트 컴포넌트가 다양한 형태를 가질수 있게 하고싶을 때
- JSX.Element
    - 리액트 엘리먼트를 prop으로 전달받아 render props 패턴으로 컴포넌트를 구현할때 활용
    
    ```tsx
    interface Props {
      icon: JSX.Element;
    }
    
    const Item = ({ icon }: Props) => {
      // prop으로 받은 컴포넌트의 props에 접근할 수 있다
      const iconSize = icon.props.size;
    
      return <li>{icon}</li>;
    };
    
    // icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다
    const App = () => {
      return <Item icon={<Icon size={14} />} />;
    };
    ```
    
    - icon prop을 JSX.Element 타입으로 선언함으로써 해당 prop에는 JSX문법만 삽입할 수 있다.
    - icon.props에 접근하여 prop으로 넘겨받은 컴포넌트의 상세한 데이터를 가져올 수 있음.
- ReactElement
    - JSX.Element 대신에 ReactElement를 사용
    - ReactElement타입을 활용하여 제네릭에 직접 해당 컴포넌트의 props 타입을 명시해줌
    - props에 접근할 때 어떤 props가 있는지 추론되어 IDE에 표시됨

## 1-7 리액트에서 기본 HTML요소 활용하기

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
    
- ComponentWithoutRef
    
    ```tsx
    type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
    type ButtonProps = {
      onClick?: NativeButtonType["onClick"];
    };
    ```
    
    - HTML 태그 속성과 호환되는 타입을 선언할 수 있음.
- 언제 ComponentWithoutRef를 사용하면 좋은지
    - 함수 컴포넌트는 전달받은 ref가 Button컴포넌트의 button 태그를 바라보지 않음.
    - 함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 ref에 기대한 값이 할당되지 않음.
