## 리액트 컴포넌트의 타입

### 클래스 컴포넌트 타입

#### 클래스 컴포넌트가 상속받는 `React.Compont` 및 `React.PureComponent` 타입 정의

```tsx
interface Component<P = {}, S = {}, SS = any> extends ComponentLifecylcle<P, S, SS> {}

class Component<P, S> {}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

- props, state, snapshot에 해당하는 타입을 제네릭으로 받음
  - `P`: props의 타입
  - `S`: state의 타입
  - `SS`: snapshot의 타입
- 상태가 있는 컴포넌트일 때는 제네릭의 두 번째 인자로 타입을 넘겨주면 상태에 대한 타입을 지정 가능

```tsx
interface WelcomeProps {
  name: string
}

class Welcome extends React.Component<WelcomeProps> {}
```

### 함수 컴포넌트 타입

#### 함수 컴포넌트에서 타입 사용하기

```tsx
// 함수 선언식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식 (1) - React.FC
const Welcome: React.FC<WelcomeProps> = ({ name }) => {}

// 함수 표현식 (2) - React.VFC
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {}

// 함수 표현식 (3) - JSX.Element를 반환 타입으로 지정
const Welcome = ({ name }: WelcomeProps): JSX.Element => {}
```

#### `@types/react`에서 `React.FC` 및 `React.VFC` 타입 정의

```tsx
// @types/react (v17)
type FC<P = {}> = FunctionComponent<P>

interface FunctionComponent<P = {}> {
  // props에 children 추가
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null
  propsTypes?: WeakValidationMap<p> | undefined
  contextTypes?: WeakValidationMap<p> | undefined
  defaultProps?: Partial<P> | undefined
  displayName?: string | undefined
}

type VFC<P = {}> = VoidFunctionComponent<P>

interface VoidFunctionComponent<P = {}> {
  (props: P, context?: any): ReactElement<any, any> | null
  propsTypes?: WeakValidationMap<p> | undefined
  contextTypes?: WeakValidationMap<p> | undefined
  defaultProps?: Partial<P> | undefined
  displayName?: string | undefined
}
```

#### `React.FC` vs `React.VFC`

- `React.FC`가 등장하고 이후 `React.VFC` 타입이 추가됨 (`@types/react` v16.9.4)
- 함수 컴포넌트 지정 시 사용하며, children 타입 허용 여부에 따라 쓰임새가 달라짐
- `React.FC`가 암묵적으로 children 포함하고 있어 children props가 필요하지 않은 컴포넌트에서 더 정확한 타입 지정을 위해 `React.VFC` 사용

> [!WARNING]

❗️리액트 v18로 넘어오면서 `React.VFC`가 삭제되고 `React.FC`에서 children이 사라짐

- FC 사용부분: https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/function_components/
- deprecatedLegacyContext: [https://legacy.reactjs.org/docs/legacy-context.html#referencing-context-in-lifecycle-methods React Docs](https://legacy.reactjs.org/docs/legacy-context.html#referencing-context-in-lifecycle-methods%20React%20Docs)
- propTypes: https://react.dev/reference/react/Component#static-proptypes
- defaultProps: https://react.dev/reference/react/Component#static-defaultprops
- defaultProps 지원 중단: https://github.com/reactjs/rfcs/pull/107 → default 파라미터 이용
  > ```tsx
  > // @types/react (v18.3.12)
  > type FC<P = {}> = FunctionComponent<P>
  >
  > interface FunctionComponent<P = {}> {
  >   // context -> deprecatedLegacyContext (deprecated)
  >   (props: P, deprecatedLegacyContext?: any): ReactNode
  >   propTypes?: WeakValidationMap<P> | undefined
  >   contextTypes?: ValidationMap<any> | undefined // deprecated
  >   defaultProps?: Partial<P> | undefined // deprecated
  >   displayName?: string | undefined
  > }
  >
  > /**
  >  * @deprecated - Equivalent to {@link React.FunctionComponent}.
  >  *
  >  * @see {@link React.FunctionComponent}
  >  * @alias {@link VoidFunctionComponent}
  >  */
  > type VFC<P = {}> = VoidFunctionComponent<P>
  >
  > /**
  >  * @deprecated - Equivalent to {@link React.FunctionComponent}.
  >  *
  >  * @see {@link React.FunctionComponent}
  >  */
  > interface VoidFunctionComponent<P = {}> {
  >   (props: P, deprecatedLegacyContext?: any): ReactNode
  >   propTypes?: WeakValidationMap<P> | undefined
  >   contextTypes?: ValidationMap<any> | undefined // deprecated
  >   defaultProps?: Partial<P> | undefined // deprecated
  >   displayName?: string | undefined
  > }
  > ```

### Children props 타입 지정

#### 보편적인 children 타입 정의

```tsx
type PropsWithChildren<P> = P & { children?: React.ReactNode | undefined }
```

#### 구체적인 chilren 타입 정의

```tsx
// 문자열 리터럴만 허용하도록 구체화
type WelcomeProps = {
  children: '천생연분' | '더 귀한 분' | '귀한 분' | '고마운 분'
}

// 문자열만 허용하도록 구체화
type WelcomeProps = {
  children: string
}

// React 엘리먼트만 허용하도록 구체화
type WelcomeProps = {
  children: React.Element
}
```

## render 메서드와 함수 컴포넌트의 반환 타입

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc48ee3d-1b55-4437-92f8-fdb7b54dff50/c917cda8-175a-42af-aa69-41e18b23fda9/image.png)

## ReactElement ([공식문서](https://ko.react.dev/reference/react/createElement#what-is-a-react-element-exactly))

```tsx
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>,
> {
  type: T
  props: P
  key: string | null
}
```

- `createElement()`의 반환 타입
  - 리액트 엘리먼트를 생성하기 위한 문법인 JSX를 통해
- 리액트는 실제 DOM이 아니라 가상의 DOM을 기반으로 렌더링하는데 가상 DOM의 엘리먼트는 ReactElement 형태로 저장됨
- 즉, `ReactElement` 타입은 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷

### [`createElement`](https://ko.react.dev/reference/react/createElement)

> JSX는 리액트 엘리먼트를 생성하기 위한 문법이며 트랜스파일러는 JSX 문법을 createElement 메서드 호출무능로 변환하여 리액트 엘리먼트 객체를 생성한다.

```tsx
const element = createElement(type, props, ...children)
```

- `type`: html 태그나 유효한 React 컴포넌트
- `props`: ref, key 등의 특수한 props를 제외한 props 객체
- `…children`(optional): 0개 이상의 자식 노드

#### 리액트 엘리먼트 객체 생성을 위한 `createElement` 호출

```tsx
const element = createElement('hi', { classname: 'greeting' }, 'Hello, world!')
```

#### `createElement`로 생성된 리액트 엘리먼트 객체

```tsx
const element = {
  type: 'hi',
  props: {
    className: 'greeting',
    children: 'Hello, world!',
  },
}
```

- 리액트는 리액트 엘리먼트 객체를 읽어서 DOM을 구성
- 리액트는 여러 개의 `createElement` 오버라이딩 메서드가 존재하는데, 이 메서드들이 반환하는 타입은 `ReactElement` 타입을 기반으로 함
- `ReactElement` 타입은 JSX의 `createElement` 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입

### 사용예시

- 컴포넌트의 prop를 `ReactElement`의 제네릭으로 지정
- `JSX.Element`가 props 타입으로 any로 지정한다면 `ReactElement`를 이용하면 타입을 더 명확하게 정의 가능
  - 타입 추론 관점에서 더 유용하게 활용 가능
  - IDE에서 props 속성 미리보기 가능

```tsx
interface IconProps {
  size: number
}

interface Props {
  icon: React.ReactElement<IconProps>
}

const Item = ({ icon }: Props) => {
  const iconSize = icon.props.size

  return <li>{icon}</li>
}

const App = () => <Item icon={<Icon size={14} />} />
```

## ReactNode

```tsx
type ReactText = string | number
type ReactChild = ReactElement | ReactText
type ReactFragment = {} | Iterable<ReactNode>

type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined
```

- 리액트에서 사용할 수 있는 모든 요소의 타입
- 리액트 render 함수가 반환할 수 있는 모든 형태 포함 → `ReactElement` 외에도 boolean, string, number 등

### 사용 예시: `props.children`

- 리액트의 Composition(합성) 모델 활용 시 prop으로 children 사용
- JSX 형태의 문법을 어떤 타입이든 children prop으로 지정할 수 있게 하고 싶을 경우

```tsx
interface MyComponentProps {
  children?: React.ReactNode
}
```

> [!TIP]

참고로 리액트 내장 타입인 `PropstWithChildren` 타입도 `ReactNode`로 children을 정의하고 있음

> ```tsx
> // @types/react (v18.3.12)
> type PropsWithChildren<P = unknown> = P & { children?: ReactNode | undefined }
> ```

## JSX.Element

> `JSX.Element`는 `ReactElement`의 특정 타입으로 props와 타입 필드를 any로 가지는 타입이다.

```tsx
declare global {
	namespace: JSX {
		interface Element extends React.ReactElement<any, any> {}
	}
}
```

- 리액트의 `ReactElement`를 확장하고 있는 타입
  - 제네릭으로 props와 타입 필드에 대해 any 타입을 가지도록 확장
- 글로벌 네임스페이스에 정의 → 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성 제공

### 사용 예시

- 리액트 엘리멘트를 prop으로 전달받아 render prop 패턴으로 컴포넌트를 구현할 때 유용하게 활용

```tsx
interface Props {
  icon: JSX.Element
}

const Item = ({ icon }: Props) => {
  const iconSize = icon.props.size

  return <li>{icon}</li>
}

const App = () => <Item icon={<Icon size={14} />} />
```

## 리액트에서 기본 HTML 요소 타입 활용하기

> HTML button 태그를 확장해서 만든 새로운 Button 컴포넌트도 기존의 button 태그가 가지고 있는 속성이나 이벤트 핸들러를 지원해야 일관성과 편의성을 모두 챙길 수 있다.

```tsx
const SquareButton = () => <button>정사각형 버튼</button>
```

- 기존 HTML 태그의 속성 타입을 활용하여 타입을 지정하는 방법
  - `DetailedHTMLProps`
  - `ComponentPropsWithoutRef`

### `DetailedHTMLProps`

```tsx
// @types/react (v18.3.12)
type DetailedHTMLProps<E extends HTMLAttributes<T>, T> = ClassAttributes<T> & E
```

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>

type ButtonProps = {
  onClick?: NativeButtonProps['onClick']
}
```

### `ComponentPropsWithoutRef`

- html 태그명을 T로 받아서 ComponentProps로 타입을 만든 다음 거기에서 PropsWithoutRef로 ref 속성을 빼준다

```tsx
// @types/react (v18.3.12)
type ComponentPropsWithoutRef<T extends ElementType> = PropsWithoutRef<ComponentProps<T>>
```

```tsx
type NativeButtonProps = React.ComponenPropstWithoutRef<'button'>

type ButtonProps = {
  onClick?: NativeButtonProps['onClick']
}
```

---

- 추가로 알아본 타입들..
  ### `PropsWithoutRef`
  ```tsx
  // Omit would not be sufficient for this. We'd like to avoid unnecessary mapping and need a distributive conditional to support unions.
  // see: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types
  // https://github.com/Microsoft/TypeScript/issues/28339
  type PropsWithoutRef<P> = P extends any ? ('ref' extends keyof P ? Omit<P, 'ref'> : P) : P

  /** Ensures that the props do not include string ref, which cannot be forwarded -> 이 버그가 PropsWithRef를 복잡하게 만들었다 */
  ```
  ### `PropsWithRef` → 복잡한 이유…. 버그 때문
  ```tsx
  type PropsWithRef<P> =
    // Note: String refs can be forwarded. We can't fix this bug without breaking a bunch of libraries now though.
    // Just "P extends { ref?: infer R }" looks sufficient, but R will infer as {} if P is {}.
    'ref' extends keyof P // ref라는 속성이 P에 존재하는지를 체크
      ? P extends { ref?: infer R | undefined } // ref가 P의 키로 존재 o -> P가 옵셔널 ref를 가졌는지 확인하고, ref 타입 infer를 이용해 R로 추론
        ? string extends R // P가 옵셔널 ref를 가짐 o -> R이 string을 포함하는 타입인지 확인
          ? PropsWithoutRef<P> & { ref?: Exclude<R, string> | undefined } // R이 string을 포함하는 타입 o -> PropsWithoutRef<P>와 ref 속성을 string을 포함하지 않는 옵셔널 속성으로 바꿈
          : P // R이 string을 포함하는 타입 x -> P 타입 반환
        : P // P가 { ref?: infer R | undefined } 에 속하지 않음 -> P 타입 반환
      : P // ref가 P의 키로 존재 x -> P 타입 반환
  ```
  ### `ComponentProps`
  ```tsx
  type ComponentProps<T extends keyof JSX.IntrinsicElements | JSXElementConstructor<any>> =
    T extends JSXElementConstructor<infer P>
      ? P
      : T extends keyof JSX.IntrinsicElements
      ? JSX.IntrinsicElements[T]
      : {}
  ```
  ### `ElementType`
  ```tsx
  type ElementType<
    P = any,
    Tag extends keyof JSX.IntrinsicElements = keyof JSX.IntrinsicElements,
  > = { [K in Tag]: P extends JSX.IntrinsicElements[K] ? K : never }[Tag] | ComponentType<P>
  ```
  ### `JSX.IntrinsicElements`
  - 타입스크립트가 JSX 문법을 사용하는 코드에서 HTML 태그와 같은 기본 요소들을 인식하고 타입을 제공하기 위해 정의된 인터페이스
  - HTML 태그 이름을 기반으로 해당 요소의 속성을 확인 및 잘못된 속성 사용이나 오타 등을 컴파일 단계에서 발견하는 역할
  - 글로벌 네임스페이스 안에 정의됨

---

### 언제 `ComponentWithoutRef`를 사용하면 좋을까?

> → ref를 사용할 때

#### 기존 button 태그의 HTML 속성을 props로 받는 `Button` 컴포넌트

- HTML button 태그를 대체하는 역할이므로 기존 button 태그의 HTML 속성을 props로 받을 수 있게 지원

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>

const Button = (props: NativeButtonProps) => <button {...props}>버튼</button>
```

> 하지만 만약 Button 컴포넌트가 ref를 props로 받는다면?

- 클래스 컴포넌트: `this.buttonRef`로 ref 생성

```tsx
class Button extends React.Component {
  constructor(ref: NativeButtonProps['ref']) {
    this.buttonRef = ref
  }

  render() {
    return <button ref={this.buttonRef}>버튼</button>
  }
}
```

- 함수 컴포넌트: `useRef` 훅으로 ref 생성

```tsx
const Button = (ref: NativeButtonPorps['ref']) => {
  const buttonRef = useRef(null)

  return <button ref={buttonRef}>버튼</button>
}
```

#### ref를 부모 컴포넌트에서 자식 컴포넌트로 넘겨줄 때 문제 발생

- 클래스 컴포넌트: `this.buttonRef`로 컴포넌트 props로 전달된 ref가 Button 컴포넌트의 button 태그를 그대로 바라볼 수 있음
  - 클래스 컴포넌트에서 ref 객체는 마운트된 컴포넌트의 인스턴스를 current 속성값으로 가짐

```tsx
class WrappedButton extends React.Component {
  constructor() {
    this.buttonRef = React.createRef()
  }

  render() {
    return (
      <div>
        <Button ref={this.buttonRef} />
      </div>
    )
  }
}
```

- 함수 컴포넌트: 전달받은 ref가 Button 컴포넌트의 button 태그를 바라보지 않음
  - 함수 컴포넌트에서 ref 객체는 생성된 인스턴스가 없어 ref에 기대한 값이 할당되지 않음

```tsx
const WrappedButton = () => {
  const buttonRef = useRef(null)

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  )
}
```

### React의 `fowardRef`

- 함수 컴포넌트에서 ref를 props로 전달받을 수 없는 문제를 해결

```tsx
const Component = fowardRef(render)
const MyInput = forwardRef(function MyInput(props, ref) {})
```

- `render`: 컴포넌트의 렌더링 함수로 props와 ref 반환 값인 JSX가 컴포넌트의 결과
  - `props`: 부모 컴포넌트가 전달한 props
  - `ref`: 부모 컴포넌트가 전달한 `ref` 속성

```tsx
const Button = forwardRef((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  )
})

const WrappedButton = () => {
  const buttonRef = useRef(null)

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  )
}
```

#### `forwardRef`의 타입 정의

> → `ComponenPropsWithoutRef` 사용!

```tsx
// button 태그에 대한 HTML 속성은 모두 포함하지만, ref 속성은 제외
type NativeButtonType = React.ComponenPropsWithoutRef<'button'>

const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  )
})
```

- 함수 컴포넌트의 props로 DetailedHTMLProps와 같이 ref를 포함하는 타입을 사용하게 되면, 실제로는 동작하지 않는 ref를 받도록 타입이 지정되어 예기치 않은 에러가 발생할 수 있음
- 따라서 HTML 속성을 확장하는 props를 설계할 떄는 `ComponenPropsWithoutRef` 타입을 사용하여 ref가 실제로 forwardRef와 함께 사용될 떄만 props로 전달되도록 타입을 정의하는 것이 안전

## 타입스크립트로 리액트 컴포넌트 만들기

- 개발 편의: 공통 컴포넌트에 어떤 타입의 속성(프로퍼티)이 제공되어야 하는지 알려줌
- 유지 보수: 필수로 전달되어야 하는 속성이 전달되지 않았을 때 에러 표시

```tsx
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = e => {
    const selected = Object.entries(options).find(([_, value]) => value === e.target.value)?.[0]
    onChange?.(selected)
  }

  return (
    <select onChange={handleChange} value={selectedOption && options[selectedOption]}>
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  )
}
```

### JSDoc로 일부 타입 지정하기

```tsx
/**
 * Select 컴포넌트
 * @param {Object} props - Select 컴포넌트를 넘겨주는 속성
 * @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
 * @param {string | undefined} props.selectedOption - 현재 선택된 option의 key 값 (optional)
 * @param {function} props.onChange - select 값이 변경되었을 때 불리는 callback 함수 (optional)
 *
 * @returns {JSX.Element}
 */
```

### props 인터페이스 적용하기

```tsx
type Option = Record<string, string>

interface SelectProps {
  options
  Option
  selectedOption?: string
  onChange?: (selected?: string) => void
}
```

### 리액트 이벤트

- 리액트는 가상 DOM을 다루면서 이벤트도 별도로 관리 → 브라우저의 고유한 이벤트와 완전히 동일하게 동작하지 않음
- 리액트는 브라우저 이벤트를 합성한 합성 이벤트(`SyntheticEvent`)를 제공

```tsx
type EventHandler<Event extends React.SyntheticEvent> = (e: Event) => void | null
type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>

const eventHandler1: GlobalEventHandlers['onchange'] = e => {
  e.target // 일반 Event는 target이 없음
}

const eventHandler2: ChangeEventHandler = e => {
  e.target // 리액트 이벤트(합성 이벤트)는 target이 없음
}
```

> [!NOTE]

#### `SyntheticEvent`

> ```tsx
> interface BaseSyntheticEvent<E = object, C = any, T = any> {
>   nativeEvent: E
>   currentTarget: C
>   target: T
>   bubbles: boolean
>   cancelable: boolean
>   defaultPrevented: boolean
>   eventPhase: number
>   isTrusted: boolean
>   preventDefault(): void
>   isDefaultPrevented(): boolean
>   stopPropagation(): void
>   isPropagationStopped(): boolean
>   persist(): void
>   timeStamp: number
>   type: string
> }
>
> interface SyntheticEvent<T = Element, E = Event>
>   extends BaseSyntheticEvent<E, EventTarget & T, EventTarget> {}
> ```

#### `handleChange`에 이벤트 타입 선언하기

- `React.EventHandler<ChangeEvent<HTMLSelectElement>>`
- `React.ChangeEventHandler<HTMLSelectElement>`

```tsx
const handleChange:React.ChangeEventHandler<HTMLSelectElement> = e => {
	const selected = Object.entries(options).find(([_, value]) => value === e.target.value?[0])
	onChange?(selected)
}
```

### 훅에 타입 추가하기

> 훅이나 외부 라이브러리 또는 내부 모듈의 함수는 **적절한 제네릭 타입을 설정**하여 활용할 수 있다.

- `useState` 훅은 제네릭으로 state 타입을 지정해줄 수 있음
- 만약 제네릭 타입을 명시하지 않으면 타입스크립트 컴파일러가 state의 초기값(default value)을 기반으로 state 타입을 추론

```tsx
const fruits = {
  apple: '사과',
  banana: '바나나',
  blueberry: '블루베리',
}

const FruitSelect: React.FC = () => {
  const [fruit, changeFruit] = useState<string | undefined>()

  return <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
}
```

#### `useState`에 타입을 지정해야 하는 이유

- 만약 타입 매개변수가 없다면 fruit의 타입이 undefined로만 추론되면서 onChagne의 타입과 일치하지 않아 오류 발생

```tsx
const [fruit, changeFruit] = useState()
// fruit: undefined
// changeFruit: (v: React.SetStateAction<undefined>) => void

// onChange: (selected?: string) => void
```

- 타입스크립트 컴파일러가 기대하지 않은 값이 들어오더라도 에러로 잡지 않아 예상치 못한 사이드 이펙트 발생

```tsx
const [fruit, changeFruit] = useState('apple') // string으로 추론

const func = () => {
  changeFruit('orange') // fruits 객체에 없는 값이 들어와도 에러 발생 x
}
```

> 이럴 때는 타입 매개변수로 좀 더 명확한 타입을 지정함으로써, state나 changeState 등을 한정된 타입으로 다룰 수 있게 강제가 필요하다.

#### `keyof typof`

- 해당 객체의 키값을 유니온 타입으로 추출하는 패턴으로 자주 사용

```tsx
type Fruit = keyof typeof fruits // 'apple' | 'banana' | 'blueberry'
const [fruit, changeFruit] = useState<Fruit | undefined>('apple')

const funct = () => {
  changeFruit('orange') // 에러 발생
}
```

### 제네릭 컴포넌트 만들기

```tsx
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>('apple')

  return <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
}
```

- selectedOption은 options에 존재하지 않는 값을 받아도 아무런 오류가 발생하지 않음
- Option의 타입에서 키(key)가 string이기만 하면 prop으로 넘겨줄 수 있기 때문
- 하지만 changeFruit의 매개변수 Fruit는 prop으로 전달해야 하는 onChange의 매개변수 타입인 string보다 좁기 때문에 타입 에러 발생
- Select를 사용하는 입장에서 제한된 키(key)와 값(value)만을 가지도록 하려면 어떻게 해야할까?
- 함수 컴포넌트 역시 함수 → 제네릭을 사용한 컴포넌트를 만들어낼 수 있음

#### 객체 형식의 타입을 받아 해당 객체의 키값을 selectedOption, onChange의 매개변수에만 적용하도록 만든 예시

```tsx
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType
  selectedOption?: keyof OptionType
  onChange?: (selected?: keyof OptionType) => void
}

const Select = <OptionType extends Record<string, string>>({
  onChange,
  options,
  selectedOption,
}: SelectProps<OptionType>) => {
  const handleChange: ChangeEventHandler<HTMLSelectElement> = e => {
    const selected = Object.entries(options).find(([_, value]) => value === e.target.value)?.[0]
    onChange?.(selected)
  }

  return (
    <select onChange={handleChange} value={selectedOption && options[selectedOption]}>
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  )
}
```

- Select 컴포넌트에 전달되는 props의 타입 기반으로 타입이 추론되어 <Select<추론된 타입>> 형태의 컴포넌트가 생성됨
  - FruitSelect에서 잘못된 selectedOption을 전달하면 타입 에러가 발생
- <Select<Fruit> … /> 형태로 작성해도 되지만 넘겨주는 props의 타입으로 타입스크립트 컴파일러가 타입 추론

### HTMLAttributes, ReactProps 적용하기

- 리액트에서 제공하는 타입을 사용하여 더 정확한 타입 설정 가능

#### `Type[’key’]`를 이용하여 객체 형식의 타입 내부 속성값 가져오기

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<'select'>

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps['id']
  className?: ReactSelectProps['className']
  // ...
}
```

#### 인터페이스의 `extends`와 유틸리티 타입 `Pick<Type, Keys>`을 이용하여 객체 형식의 타입 내부 속성값 추출하기

```tsx
interface SelectProps<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, 'id' | 'className'> {
  // ...
}
```

### styled-components를 활용한 스타일 정의

#### theme 객체 및 타입 정의

```tsx
const theme = {
  fontSize: {
    default: '16px',
    small: '14px',
    large: '18px',
  },
  color: {
    white: '#FFF',
    black: '#000',
  },
}

type Theme = typeof theme
type Color = keyof Theme['color']
type FontSize = keyof Theme['fontSize']
```

#### 스타일 컴포넌트의 props 및 스타일 속성 타입 정의하고 사용하기

- 이때 유틸리티 타입 `Partial<Type>`을 사용하면 객체 형식의 타입 내 모든 속성을 옵셔널로 설정 가능

```tsx
interface SelectStyleProps extends Partial<SelectStyleProps> {
  color: Color
  fontSize: FontSize
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`

const Select = ({ fontSize = 'default', color = 'black' }) => {
  return <StyledSelect fontSize={fontSize} color={color}></StyledSelect>
}
```

### 공변성과 반공변성

#### 객체의 메서드 타입을 정의하는 방법

-

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void // 에러 발생
  onChangeB?(selected: T): void
}

const Component = () => {
  const changeToPineApple = (selectedApple: 'apple') => {
    console.log('this is pine' + selectedApple)
  }

  return <Select onChangeA={changeToPineApple} onChangeB={changeToPineApple} />
}
```

```tsx
interface User {
  id: string
}

interface Member extends User {
  nickName: string
}

let users: Array<User> = []
let members: Array<Member> = []

users = members
members = users // 에러 발생
```

- 일반적인 타입들은 공변성을 가지고 있어서 좁은 타입에서 넓은 타입으로 할당이 가능
- 하지만 제네릭 타입을 지닌 함수는 반공변성을 가짐
- 즉, T<B>가 T<A>의 서브타입이 되어, 좁은 타입 T<A>의 함수를 넓은 타입 T<B>의 함수에 적용할 수 없음

```tsx
type PrintUserInfo<U extends User> = (user: U) => void

let printUser: PrintUserInfo<User> = user => console.log(user.id)
let printMember: PrintUserInfo<Member> = user => console.log(user.id, user.nickName)

printMember = printUser
printUser = printMember // 에러 발생
```

- printMember 타입을 가진 함수는 Member 타입의 객체로부터 nickName까지 함께 출력하는 역할을 함
- printUser는 PrintUserInfo<User> 타입으로 정의되어 있어서 Member 타입을 매개변수로 받을 수 없는 상황
- printMember 함수를 printUser 변수에 할당할 수 없음

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void
  onChangeB?(selected: T): void
}
```

- onChangeA같이 함수 타입을 화살표 표기법으로 작성하게 되면 반공변성을 띔
- onChagneB같이 함수 타입을 지정하면 공변성과 반공변성을 모두 가지는 이변성을 띔
- **안전한 타입 가드를 위해서는 일반적으로 반공변적인 함수 타입을 설정을 권장**

## 최종 코드

```tsx
import React, { useState } from 'react'
import styled from 'styled-components'

const theme = {
  fontSize: {
    default: '16px',
    small: '14px',
    large: '18px',
  },
  color: {
    white: '#FFFFFF',
    black: '#000000',
  },
}

type Theme = typeof theme

type FontSize = keyof Theme['fontSize']
type Color = keyof Theme['color']

interface SelectStyleProps {
  color: Color
  fontSize: FontSize
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`

type ReactSelectProps = React.ComponentPropsWithoutRef<'select'>

interface SelectProps<OptionType extends Record<string, string>> extends Partial<SelectStyleProps> {
  id?: ReactSelectProps['id']
  className?: ReactSelectProps['className']
  options: OptionType
  selectedOption?: keyof OptionType
  onChange?: (selected?: keyof OptionType) => void
}

const Select = <OptionType extends Record<string, string>>({
  className,
  id,
  options,
  onChange,
  selectedOption,
  fontSize = 'default',
  color = 'black',
}: SelectProps<OptionType>) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = e => {
    const selected = Object.entries(options).find(([_, value]) => value === e.target.value)?.[0]
    onChange?.(selected)
  }

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
  )
}

const fruits = { apple: '사과', banana: '바나나', blueberry: '블루베리' }

type Fruit = keyof typeof fruits

const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>()

  return (
    <Select
      className="fruitSelectBox"
      options={fruits}
      onChange={changeFruit}
      selectedOption={fruit}
      fontSize="large"
    />
  )
}

export default FruitSelect
```
