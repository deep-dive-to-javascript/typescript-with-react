## 8장 JSX에서 TSX로
### 8.1 리액트 컴포넌트의 타입
1. 클래스 컴포넌트 타입
```tsx
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
  /* ... 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

- 클래스 컴포넌트가 상속받는 타입 정의는 위와 같으며 P와 S는 각각 props와 state의 타입을 의미

2. 함수 컴포넌트 타입
```ts
// 함수 선언을 사용한 방식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식을 사용한 방식 - React.FC 사용
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - React.VFC 사용
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - JSX.Element를 반환 타입으로 지정
const Welcome = ({ name }: WelcomeProps): JSX.Element => {};

type FC<P = {}> = FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  // props에 children을 추가
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}

type VFC<P = {}> = VoidFunctionComponent<P>;

interface VoidFunctionComponent<P = {}> {
  // children 없음
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

- React 18로 넘어오면서 React.VFC가 사라지고 React.FC에서 children이 사라짐

3. Children props 타입 지정
```ts
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```
- 가장 보편적인 children props 타입 지정 방법
- ReactNode는 ReactElement외에도 boolean, number 등 여러 타입을 포함하고 있는 타입으로 더 구체적으로 타이핑하는 용도에는 적합하지 않음

4. render 메서드와 함수 컴포넌트의 반환 타입 - React.ReactElement vs JSX.Element vs React.ReactNode

**React.ReactElement**
```ts
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
- React.createElement로 생성된 React 엘리먼트를 나타내는 타입
- 실제 DOM이 아닌 가상의 DOM 엘리먼트를 나타냄
- 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷

**JSX.Element**
```ts
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```
- JSX.Element는 리액트의 ReactElement를 확장하고 있는 타입
- 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있는 유연성 제공

**React.ReactNode**
```ts
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
- 단순히 ReactElement 외에도 boolean, string, number 등의 여러 타입을 포함

5. ReactElement, ReactNode, JSX.Element 활용하기

**ReactElement**
- 앞에서 살펴본 것과 같이 React.createElement로 생성된 React 엘리먼트를 나타내는 타입
- JSX는 createElement 메서드를 호출하기 위한 문법
- 트랜스파일러는 JSX 문법을 createElement 호출로 변환하여 리액트 엘리먼트를 생성
```ts
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
- 리액트는 이런 식으로 만들어진 리액트 엘리먼트 객체를 읽어서 DOM을 구성
- 리액트는 여러 개의 createElement 오버라이딩 메서드가 존재하는데, 이 메서드들이 반환하는 타입은 ReactElement 타입을 기반으로 함

**ReactNode**
- ReactNode 타입에 대해 알아보기 전에 먼저 ReactChild 타입을 살펴보자
```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
```
- ReactChild 타입은 ReactElement 보다는 좀 더 넓은 범위를 갖고 있음

```ts
//ReactNode
type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태
type ReactNode =
    | ReactChild
    | ReactFragment
    | ReactPortal
    | boolean
    | null
    | undefined;
```
- ReactNode는 리액트의 render 함수가 변환할 수 있는 모든 형태를 담고 있음

**JSX.Element**
```declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```
- JSX.Element는 ReactElement의 제네릭으로 props와 타입 필드에 대해 any 타입을 가지도록 확장하고 있음
- JSX.Element는 ReactElement의 특정 타입으로 props와 타입 필드를 any로 가지는 타입

6. 사용예시

**ReactNode**
- ReactNode 타입은 리액트 컴포넌트가 가질 수 있는 모든 타입을 의미
- JSX 형태의 문법을 어떤 타입이든 받을 수 있게 하고 싶다면 ReactNode를 사용하면 된다
- ReactNode는 prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고 싶을 때 유용하게 사용됨
```ts
type PropsWithChildren<P = unknown> = P & {
  children?: ReactNode | undefined;
};

interface MyProps {
  // ...
}

type MyComponentProps = PropsWithChildren<MyProps>;
```

**JSX.Element**
- JSX.Element는 앞서 언급한대로 props와 타입 필드가 any 타입인 리액트 엘리먼트를 나타냄
- 이러한 특성 때문에 리액트 엘리먼트를 prop으로 전달받아 render props 패턴으로 컴포넌트를 구현할 때 유용하게 활용 가능
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

**ReactElement**
- 앞서 살펴본 JSX.ELement 예시를 확장하여 추론 관점에서 더 유용하게 활용할 수 있는 방법

```tsx
interface IconProps {
  size: number;
}

interface Props {
  // ReactElement의 props 타입으로 IconProps 타입 지정
  icon: React.ReactElement<IconProps>;
}

const Item = ({ icon }: Props) => {
  // icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론된다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};
```

7. 리액트에서 기본 HTML 요소 타입 활용하기

**DetailedHTMLProps와 ComponentWithoutRef**
```ts
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};
```

**React.ComponentPropsWithoutRef**
```ts
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```

**언제 ComponentPropsWithoutRef를 사용하면 좋을까**
- 아래와 같이 커스텀 버튼 컴포넌트를 만든다고 가정해보자
```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
  return <button {...props}>버튼</button>;
};
```
- 이때 ref를 Props로 받을 경우 고려해야 할 사항이 있다.
- 클래스 컴포넌트와 함수 컴포넌트에서 ref를 props로 받아 전달하는 방식에 차이가 있다
```tsx
// 클래스 컴포넌트
class Button extends React.Component {
  constructor(props) {
    super(props);
    this.buttonRef = React.createRef();
  }

  render() {
    return <button ref={this.buttonRef}>버튼</button>;
  }
}

// 함수 컴포넌트
function Button(props) {
  const buttonRef = useRef(null);

  return <button ref={buttonRef}>버튼</button>;
}
```
- 클래스 컴포넌트로 만들어진 버튼은 컴포넌트 Props로 전달된 ref가 Button 컴포넌트의 button 태그를 그대로 바라보게 된다.
- 하지만 함수 컴포넌트의 경우 기대와 달리 전달받은 ref가 Button 컴포넌트의 button 태그를 바라보지 않는다.
- 클래스 컴포넌트에서 ref 객체는 마운트된 컴포넌트의 인스턴스를 current 속성값으로 가지지만, 함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 ref에 기대한 값이 할당되지 않는 것이다
- 이러한 차이로 인해 함수 컴포넌트에서 ref를 사용할 때는 React.forwardRef를 사용해야 한다
```tsx
// forwardRef를 사용해 ref를 전달받을 수 있도록 구현
const Button = forwardRef((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});

// buttonRef가 Button 컴포넌트의 button 태그를 바라볼 수 있다
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  );
};
```

- 앞 코드에서 Button 컴포넌트에 대한 타입선언을 할 때 ComponentPropsWithoutRef로 선언하고 forwardRef에서 Ref에 대한 타입 정의를 하게하여 예기치 않은 에러를 방지할 수 있다
```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef의 제네릭 인자를 통해 ref에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의했다
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```

### 8.2 타입스크립트로 리액트 컴포넌트 만들기
1. JSX로 구현된 Select 컴포넌트
- 추가적인 설명이 없다면 컴포넌트를 사용하는 입장에서 각 속성에 어떤 타입의 값을 전달해야 할지 알기 어렵다
```jsx
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
2. JSDocs로 일부 타입 지정하기
- 컴포넌트의 속성 타입을 명시하기 위해 JSDocs를 사용할 수 있음
```jsx
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

3. props 인터페이스 적용하기
- jsdocs를 사용하면 각 속성의 대략적인 타입과 어떤 역할을 하는지 파악할 수 있지만 Options가 어떤 형식의 객체를 나타내는지나 매개변수 반환값에 대한 구체적인 정보를 알기 어렵다
- jsx를 tsx 확장자로 변경한 후에 props에 대한 인터페이스를 만들어 타입을 명시할 수 있다
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

4. 리액트 이벤트
- 리액트는 가상 DOM을 다루면서 이벤트도 별도로 관리한다.
- onClick, onChange 같이 DOM 엘리먼트에 등록되어 처리하는 이벤트 리스너와 달리, 리액트 컴포넌트에 등록되는 이벤트 리스너는 onClick, onChange처럼 카멜 케이스로 표기한다.
- 따라서 리액트 이벤트는 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지는 않음
- 예를 들어 리액트 이벤트 핸들러는 이벤트 버블링 단게에서 호출된다. 이벤트 캡쳐 단계에서 이벤트 핸들러를 등록하기 위해서는 onCaptureClick과 같이 onCapture 접두사를 사용해야 한다
- 리액트는 브라우저 이벤트를 합성한 합성 이벤트(SyntheticEvent)를 제공한다. 
```tsx
type EventHandler<Event extends React.SyntheticEvent> = (
  e: Event
) => void | null;
type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>;

const eventHandler1: GlobalEventHandlers["onchange"] = (e) => {
  e.target; // 일반 Event는 target이 없음
};

const eventHandler2: ChangeEventHandler = (e) => {
  e.target; // 리액트 이벤트(합성 이벤트)는 target이 있음
};

const Select = ({ onChange, options, selectedOption }: SelectProps) => {
    const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
        const selected = Object.entries(options).find(
            ([_, value]) => value === e.target.value
        )?.[0];
        onChange?.(selected);
    };

    return <select onChange={handleChange}>{/* ... */}</select>;
};
```

5. 훅에 타입 추가하기
- useState 같은 함수 역시 타입 매개변수를 지정해줌으로써 반환되는 state 타입을 지정해줄 수 있다
- 만약 제네릭 타입을 명시하지 않으면 타입스크립트 컴파일러는 초깃값의 타입을 기반으로 State 타입을 추론한다
```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
  );
};
```
- 만약 타입 매개변수가 없다면 Fruit의 타입이 Undefined로만 추론되면서 onChange의 타입과 일치하지 않아 오류 발생
```tsx
// fruit: undefined;
// changeFruit: (v: React.SetStateAction<undefined>) => void;
const [fruit, changeFruit] = useState();

return (
  <Select
    // Error - SetStateAction<undefined>와 맞지 않음
    // (changeFruit에는 undefined만 매개변수로 넘길 수 있음)
    onChange={changeFruit}
    options={fruits}
    selectedOption={fruit}
  />
);
```
- 작성자는 fruit가 반드시 apple, banana, blueberry 중 하나라고 기대하고 있지만 제네릭 타입을 지정해주지 않는다면 타입스크립트 컴파이일러는 fruit를 string으로 추론할 것이고 예상치못한 sideEffect가 발생할 수 있다
- 따라서 타입 매개변수로 좀 더 명확한 타입을 지정하는 것이 좋다
```tsx
type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
const [fruit, changeFruit] = useState<Fruit | undefined>("apple");

// 에러 발생
const func = () => {
  changeFruit("orange");
};
```

6. 제네릭 컴포넌트 만들기
```tsx
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption="orange" />
  );
};
```
- selectedOption은 options에 존재하지 않는 값을 받아도 아무런 오류가 발생하지 않음
- key가 string이기만 하면 prop으로 넘겨줄 수 있기 때문
- 제네릭을 사용한 컴포넌트를 만들어 option의 타입을 제한시켜줄 수 있음
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

- 이제 잘못된 selectOption을 전달하면 타입 에러가 발생한다
```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  // ...
  // <Select<Fruit> ... />으로 작성해도 되지만, 넘겨주는 props의 타입으로 타입 추론을 해줍니다
  // Type Error - Type "orange" is not assignable to type "apple" | "banana" | "blueberry" | undefined

  return (
    <Select options={fruits} onChange={changeFruit} selectedOption="orange" />
  );
};
```

7. HTMLAttributes, ReactProps 적용하기
- className, id와 같은 리액트 컴포넌트의 기본 Props를 추가하려면 SelectProps에 직접 className?: string, id?: string;을 넣어도 되지만 아래처럼 리액트에서 제공하는 타입을 사용하면 더 정확한 타입을 설정할 수 있다

```ts
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  // ...
}
```

- Pick 키워드를 활용하여 아래처럼 사용할 수도 있다.

```ts
interface SelectProps<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, "id" | "key" | /* ... */> {
  // ...
}
```

8. styled-components를 활용한 스타일 정의
```ts
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
```

- Select를 사용하는 부모 컴포넌트에서 원하는 스타일을 적용하기 위해 Select 컴포넌트의 props에 SelectStyleProps 타입을 상속한다
- Partial<Type>을 사용하면 객체 형식의 타입 내 모든 속성이 옵셔널로 설정된다
```tsx
interface SelectProps extends Partial<SelectStyleProps> {
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize = "default",
  color = "black",
}: // ...
SelectProps<OptionType>) => {
  // ...

  return (
    <StyledSelect
      // ...
      fontSize={fontSize}
      color={color}
      // ...
    />
  );
};
```

9. 공변성과 반공변성
- 객체의 메서드 타입을 정의하는 방법은 2가지가 있다.

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}

const Component = () => {
  const changeToPineApple = (selectedApple: "apple") => {
    console.log("this is pine" + selectedApple);
  };

  return (
    <Select
      // Error
      // onChangeA={changeToPineApple}
      // OK
      onChangeB={changeToPineApple}
    />
  );
};
```
- onChangeA일때는 타입에러가 발생하지만 onChangeBV일때는 타입 에러가 발생하지 않는다
- 이유를 알아보기 위해 다른 예시를 살펴보겠다
- 아래 예시에서는 모든 User가 Id를 가지고 있으며 회원은 회원 가입시 등록한 별명을 추가로 갖고 있다
- Member는 User를 상속하고 있는데 User보다 더 좁은 타입이자 User의 서브타입이다
- 타입 A가 B의 서브타입일 때, `T<A>`가 `T<B>`의 서브타입이 된다면 공변성을 띠고 있다고 말한다

```ts
// 모든 유저(회원, 비회원)은 id를 갖고 있음
interface User {
  id: string;
}

interface Member extends User {
  nickName: string;
}

let users: Array<User> = [];
let members: Array<Member> = [];

users = members; // OK
members = users; // Error
```
- 일반적인 타입들은 공변성을 가지고 있어서 좁은 타입에서 넓은 타입으로 할당이 가능하다
- 하지만 제네릭 타입을 지닌 함수는 반공변성을 가진다. 즉 `T<B>`가 `T<A>`의 서브타입이 되어, 좁은 타입 `T<A>`의 함수를 넓은 타입 `T<B>`의 함수에 적용할 수 없다는 것을 의미한다
```tsx
type PrintUserInfo<U extends User> = (user: U) => void;

let printUser: PrintUserInfo<User> = (user) => console.log(user.id);

let printMember: PrintUserInfo<Member> = (user) => {
  console.log(user.id, user.nickName);
};

printMember = printUser; // OK.
printUser = printMember; // Error - Property 'nickName' is missing in type 'User' but required in type 'Member'.
```

- 위 예시에서 볼 수 있듯이 printMember 타입을 가진 함수는 Member 타입의 객체부터 nickName까지 함께 출력하는 역할을 함
- printUser는 PrintUserInfo<User> 타입으로 정의되어 있어서 Member 타입을 매개변수로 받을 수 없는 상황
- 따라서 printMember 함수를 printUser 변수에 할당할 수 없다

```ts
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}
```

- 첫번째 예시로 돌아가 --strict 모드에서 onChangeA와 같이 함수 타입을 화살표 표기법으로 작성한다면 반공변성을 띠게 된다.
- 또한 OnChangeB와 같이 함수 타입을 지정하면 공변성과 반공변성을 모두 가지는 이변성을 띠게 된다.
- 안전한 타입 가드를 위해서는 특수한 경우를 제외하고는 일반적으로 반공변적인 함수 타입을 설정하는 것이 권장됨


### 활용
- ReactNode와 ReactElement 
- DetailedHTMLProps와 ComponentWithoutRef