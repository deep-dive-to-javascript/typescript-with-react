# 11장 CSS-in-JS

2024.12.01 dasom



## 11.1 CSS-in-JS란

### 1. CSS-in-JS와 인라인 스타일의 차이

> 인라인 스타일?
>
> HTML 요소 내부에 직접 스타일을 적용하는 방식을 말한다. HTML 태그의 style 속성을 사용하여 인라인 스타일을 적용할 수 있다.

* 인라인 스타일

```ts
const textStyles = {
	color: white;
  backgroundColor: black;
}

const MyComponent = () => {
  return (
  	<p style={textStyles}>hi!</p>
  )
}

// 위 코드는 브라우저에서 DOM 노드를 아래와 같이 연결 (노드에 속성으로 스타일 추가)
<p style="color: white; background-color: black;">hi!</p>
```



* CSS-in-JS

```ts
import styled from "styled-components";

const Text = styled.div`
	color: white;
  background: black;
`

const Example = () => <Text>hi?</Text>

// 위 코드는 브라우저에서 DOM 노드를 아래와 같이 연결 (DOM상단에 style태그 추가)
<style>
  .hash136s1 {
		background-color: black;
		color: white;
}
</style>

<p class="hash136s1">hi?</p>
```



* CSS-in-JS의 장점
  * CSS-in-JS는 실제로 CSS가 생성되기 때문에 미디어 쿼리, 수도 선택자 등과 같은 CSS 기능을 누리기 쉬움
  * 컴포넌트로 생각할 수 있음 (CSS-in-JS는 스타일을 컴포넌트 단위로 추상화)
  * 부모와 분리할 수 있음
  * 스코프를 가짐
  * 자동으로 벤더 프리픽스가 붙음 (특정 브라우저에서 제대로 동작하도록 하기 위해 추가되는 접두사)
  * 자바스크립트와 CSS 사이에 상수와 함수를 쉽게 공유할 수 있음

 

### 2. CSS-in-JS 등장 배경

스타일링 라이브러리의 구분

* CSS Preprocessor
  * sass/scss
  * less
  * stylus
* CSS in JS
  * styled-components
  * emotion

CSS가 웹 어플리케이션의 UI를 설계하는 데에도 사용됨에 따라, 웹 환경이 변하면서 복잡도가 같이 증가하게 되었다.이에 따라 제기된 CSS의 대표적인 문제점 7가지는 아래와 같다.

* 글로벌 네임스페이스
* 의존성: CSS의 의존성과 자바스크립트의 의존성이 달라서 생성되는 문제
* 불필요한 코드 제거: 불필요한 CSS를 찾아 제거하기 어려움
* 최소화: 클래스 이름 최소화의 어려움
* 상수 공유: 자바스크립트와 상태 값을 공유할 수 없음
* 비결정적 해결: CSS 로드 순서에따라 스타일 우선순위가 달라짐
* 고립: CSS의 외부 수정을 관리하기 어려움



이에 따라 CSS-in-JS 라이브러리가 등장하게 되었고, 스타일이라는 요소를 컴포넌트의 일부로 간주할 수 있게 되었다.



### 3. CSS-in-JS 사용하기

추가 예시 (by ChatGPT)

* Styled

```ts
/** @jsxImportSource @emotion/react */
import styled from '@emotion/styled';

const Button = styled.button`
  background-color: #3498db;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  &:hover {
    background-color: #2980b9;
  }
`;

const App = () => (
  <div>
    <Button>Click Me</Button>
  </div>
);

export default App;
```

* css

```ts
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';

const buttonStyle = css`
  background-color: #e74c3c;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  &:hover {
    background-color: #c0392b;
  }
`;

const App = () => (
  <div>
    <button css={buttonStyle}>Click Me</button>
  </div>
);

export default App;
```

* Props

```ts
const Button = styled.button`
  background-color: ${({ primary }) => (primary ? '#2ecc71' : '#e74c3c')};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  &:hover {
    background-color: ${({ primary }) => (primary ? '#27ae60' : '#c0392b')};
  }
`;

const App = () => (
  <div>
    <Button primary>Primary Button</Button>
    <Button>Secondary Button</Button>
  </div>
);
```





## 11.2 유틸리티 함수를 활용하여 styled-components의 중복 타입 선언 피하기

### 1. props타입과 styled-components 타입의 중복 선언 및 문제점

컴포넌트 props와 styled-components에서의 props의 값이 동일할 때에도 각 props를 따로 정의해야하는 점 때문에 코드 중복이 발생한다. 이를 해결하기 위해 Pick, Omit과 같은 유틸리티 타입을 활용할 수 있다.

```ts
const HrComponent = styled.hr<Pick<Props, "height" | "color">>`
...`;
```







