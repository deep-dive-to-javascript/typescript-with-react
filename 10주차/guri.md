# 11장
# 1. CSS-in-JS란

## 1.1 CSS-in-JS와 인라인 스타일의 차이

- CSS-in-JS
    - CSS-in-CSS보다 더 강력한 추상화 수준을 제공
    - 자바스크립트로 스타일을 선언적이고 유지보수 가능한 방식으로 표현
- 인라인 스타일과의 차이
    - 인라인 스타일
    
    ```tsx
    const textStyles = {
      color: 'white',
      backgroundColor: 'black'
    }
    
    const SomeComponent = () => {
      return (
        <p style={textStyles}>inline style!</p>
      );
    };
    
    ```
    
    ```tsx
    <p style="color: white; background-color: black;">inline style!</p>
    ```
    
    - CSS-in-JS
    
    ```tsx
    import styled from "styled-components";
    
    const Text = styled.div`
      color: white;
      background: black;
    `;
    
    // 다음처럼 사용
    const Example = () => <Text>Hello CSS-in-JS</Text>;
    ```
    
    ```tsx
    <style>
      .hash136s21 {
        background-color: black;
        color: white;
      }
    </style>
    
    <p class=”hash136s21”>Hello CSS-in-JS</p>
    ```
    
    - 인라인 스타일은 DOM노드에 속성으로 스타일을 추가하는 반면, CSS-in-JS는 DOM상단에 <style>태그를 추가함.
    - CSS-in-JS는 실제로 CSS스타일을 생성하기 때문에 미디어 쿼리, 슈도 선택자 등과 같은 CSS기능을 손쉽게 누릴 수 있다.
- CSS-in-JS의 장점
    1. **컴포넌트로 생각할 수 있음** : 스타일을 컴포넌트 단위로 추상화하여 생각할 수 있게 해줌. 별도의 스타일시트를 유지보수할 필요 없이 각 컴포넌트의 스타일을 관리할 수 있음
    2. **부모와 분리할 수 있음** : CSS에는 명시적으로 정의하지 않은 경우 부모 요소에서 자동으로 상속되는 속성이 있음. 하지만 CSS-in-JS는 이러한 상속을 받지 않음. 따라서 컴포넌트의 스타일은 부모와 독립하여 독립적으로 동작함
    3. **스코프를 가짐** : CSS는 하나의 전역 네임스페이스를 가지기 때문에 충돌을 피하기 어려움 하지만 CSS-in-JS는 CSS로 컴파일할 때 고유의 이름을 생성하여 스코프를 만들어줌. 따라서 선택자 충돌을 방지할 수 있음
    4. **자동으로 벤더 프리픽스가 붙음** : CSS-in-JS는 자동으로 벤더 프리픽스를 추가하여 브라우저 호환성을 향상해줌
    5. **자바스크림트와 CSS사이에 상수와 함수를 쉽게 공유할 수 있음** : CSS-in-JS를 활용하면 자바스크립트 변수, 상수, 함수를 스타일 코드 내에서 쉽게 사용할 수 있음.

## 1.2 CSS-in-JS 등장 배경

- 스타일링 라이브러리
    - CSS Preprocessor
        - sass/scss
        - leff
        - stylus
    - CSS in JS
        - styled-components
        - emotion
- 규모가 크고 동적인 웹 애플리케이션을 유지보수하기위해 해결해야할 CSS의 문제점
    1. 글로벌 네임스페이스 : 모든 스타일이 전역 공간을 공유하므로 중복되지 않는 CSS 클래스 이름을 고민해야함.
    2. 의존성 : CSS의 의존성과 자바스크립트의 의존성이 달라서 사용하지 않는 스타일이 포함되거나 필요한 스타일이 누락되는 문제가 발생함(현재는 번들러의 발전으로 거의 해결)
    3. 불필요한 코드 제거 : 기능 추가, 수정, 삭제 과정에서 불필요한 CSS를 삭제하기 어려움
    4. 최소화 : 클래스 이름을 최소화하기 어려움
    5. 상수 공유 : 자바스크립트와 상태 값을 공유할 수 없음. (현재는 CSS Variable이 도입되어 CSS의 공식 기능으로 제공)
    6. 비결정적 해결: CSS 로드 순서에 따라 스타일 우선순위가 달라짐
    7. 고립 : CSS의 외부 수정을 관리하기 어려움.
- 다음과 같은 단점을 해결하기 위해 CSS-in-JS라이브러리가 등장
- CSS-in-JS를 통해 스타일 요소를 컴포넌트의 일부로 포함할 수 있게됨.

## 1.3 CSS-in-JS사용하기

```tsx

import styled from "@emotion/styled";

export const Button = styled.button<{ primary: boolean }>`
  background: transparent;
  border: none;
  cursor: pointer;
  font-size: inherit;
  padding: 0;
  margin: 0;
  color: ${({ primary }) => (primary ? "red" : "blue")};
`;
```

- 템플릿 리터럴을 활용해서 동적인 스타일을 정의하면 됨
    - props의 타입을 정의하고 이 props를 활용해서 동적인 스타일을 구현함.
- variant props의 유형에 따라 다른 스타일을 적용하고 싶은 경우 css함수를 사용해 스타일을 정의하고 variant값에 따라 맵 객체를 생성하여 사용할 수 있음

```tsx
import { css, SerializedStyles } from "@emotion/react";
import styled from "@emotion/styled";

type ButtonRadius = "xs" | "s" | "m" | "l";

export const buttonRadiusStyleMap: Record<ButtonRadius, SerializedStyles> = {
  xs: css`
    border-radius: ${radius.extra_small};
  `,
  s: css`
    border-radius: ${radius.small};
  `,
  m: css`
    border-radius: ${radius.medium};
  `,
  l: css`
    border-radius: ${radius.large};
  `,
  };

export const Button = styled.button<{ radius: string }>`
  ${({ radius }) => css`
    /* ...기타 스타일은 생략 */
    ${buttonRadiusStyleMap[radius]}
  `}
`;
```

- RoundButton, SquareButton등 여러 버튼 컴포넌트를 구현해야하는 경우 공통적인 버튼 스타일을 따로 정의한 다음에 각 컴포넌트 스타일에서 이를 확장하여 구현할 수 있음.

```tsx
const RoundButton = styled(CommonButton)``;
const SquareButton = styled(CommonButton)``;
```


# 2. 유틸리티 함수를 활용하여 styled-components의 중복 타입 선언 피하기

- styled-component에 넘겨주는 props의 타입을 styled-component에서도 정의해줘야함. 이때 Pick, Omit같은 유틸리티 타입을 유용하게 활용할 수 있음.

## 2.1 props타입과 styled-components타입의 중복 선언 문제점

- 아래 컴포넌트는 수평선을 그어주는 HrComponent 임.
- HrComponent의 props중 height, color, isFull속성은 styled-component인 HrComponent에 그대로 prop으로 전달되기 때문에 타입이 같음. 여기서는 아래 코드처럼  height, color, isFull에 대한 StyledProps타입을 새로 정의했음.
- 이때 코드의 중복이 발생하고 props의 타입이 변경되면 StyledProps의 타입도 변경해야함.

```tsx
interface Props {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
  // ...
}

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  // ...
  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  ); 
};

interface StyledProps {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
};

const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) => height || "10px"};
  margin: 0;
  background-color: ${({ color }) => colors[color || "gray7"]};
  border: none;
  
  ${({ isFull }) =>
    isFull &&
    css`
      margin: 0 -15px;
    `}
`;
```

- 다음과 같이 Pick이나 Omit같은 유틸리티 타입을 활용하면 중복되는 타입을 피할 수 있어 유지보수 측면에서 긍정적이다.

```tsx
const HrComponent = styled.hr<Pick<Props, "height" | "color" | "isFull">>`
  // ...
`;
```

# 12장

# 1. 앰비언트 타입 활용하기

## 1.1 앰비언트 타입 선언

- .d.ts확장자를 가진 파일에서도 타입 선언을 할 수 있음
- 앰비언트 타입 선언
    - .d.ts 확장자 파일에서는 타입 선언만 할 수 있으며 값을 표현할 수 없음.
    - 앰비언트 타입 선언 : 값을 포함하는 일반적인 선언과 구별되는 .d.ts 확장자를 가진 파일에서 하는 타입 선언
    - 앰비언트 타입 선언으로 값을 정의할 수 없지만 declare라는 키워드를 사용해 어딘가에 자바스크립트 값이 존재한다는 사실을 선언할 수 있음.
- 앰비언트 타입 활용 사례
    - 타입스크립트에서 .js .ts형식이 아닌 파일을 임포트할때 에러가 발생하는 경우가 있음.
    - 타입스크립트는 .ts와 .js파일만 이해하며 그 외 다른 파일 형식은 인지하지 못함. 따라서 이를 모듈로 가져오려고 하면 에러가 발생함.
    - decalre 키워드를 사용하면서 특정형식을 모듈로 선언하면 타입스크립트 컴파일러에 미리 정보를 제공하면서 에러를 수정할 수 있다.
- 자바스크립트로 작성된 라이브러리
    - 자바스크립트로 작성된 npm라이브러리가 있다고 가정. 자바스크립트로 작성되었기 때문에 타입 선언이 없음. 따라서 타입이 any로 추론됨. tsconfig.json에서 any를 사용하지 못하게 설정했다면 프로젝트가 빌드되지 않을것.
    - 이때 앰비언트 타입 선언을 사용할 수 있음.
- 타입스크립트로 적성된 라이브러리
    - 타입스크립트 파일을 직접 배포하여 라이브러리 사용자가 타입스크립트를 컴파일할 때 라이브러리 코드도 함께 컴파일 하게 할 수도 있다. 그러나 자바스크립트 파일과 .d.ts파일로 배포하면 라이브러리 코드를 따로 컴파일하지 않아도 되기 때문에 컴파일 시간이 단축된다.
- 자바스크립트 어딘가에 전역 변수가 정의되어있음을 타입스크립트에 알릴 때
    - 타입스크립트로 직접 구현하지 않았지만 실제 자바스크립트 어딘가에 전역 변수가 정의되어있는 상황을 타입스크립트에 알릴 때 앰비언트 타입 선언을 사용함.

## 1.2 앰비언트 타입 선언시 주의점

- 타입스크립트로 만드는 라이브러리에는 불필요
    - tsconfig.json의 declaration을 true로 설정하면 타입스크립트 컴파일러가 .d.ts파일을 자동으로 생성함. 수동으로 작성할 필요가 없음.
- 전역으로 타입을 정의하여 사용할 때 주의할 점
    - 서로 다른 라이브러리에서 동일한 이름의 앰비언트 타입 선언을 하면 충돌이 발생
    - 앰비언트 타입 선언은 명시적인 임포트나 익스포트가 없어 코드의 의존성 관계가 명확하지 않아 나중에 변경시 어려움이 있음.

## 1.3 앰비언트 타입 선언을 잘못 사용했을 때 문제점

- .ts파일 내의 앰비언트 변수 선언은 혼란을 야기함.
    - 앰비언트 타입의 의존성 관계가 보이지 않으므로 영향범위 파악이 어려움.
