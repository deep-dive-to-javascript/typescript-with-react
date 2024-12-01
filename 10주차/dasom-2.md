# 12장 타입스크립트 프로젝트관리

2024.12.01 dasom



## 12.1 앰비언트 타입 활용하기

### 1. 앰비언트 타입 선언

> 앰비언트(ambient)는 사전적으로 '주변의'라는 의미를 가지며, 앰비언트 타입 선언이란 값을 포함하는 일반적 선언과 구별하기 위해 .d.ts 확장자를 가진 파일에서 하는 타입 선언을 말한다.

앰비언트 타입 선언으로 값을 정의할 수는 없지만, declare라는 키워드를 사용하여 어딘가에 자바스크립트 값이 존재한다는 사실을 선언할 수 있다.



**대표적인 앰비언트 타입 선언 활용 사례**

타입스크립트는 기본적으로 .ts와 .js 파일만 이해하며 그 외 다른 파일 형식 인식하지 못한다. 이런 상황에서 타입스크립트의 declare 키워드를 사용해 특정 형식을 모듈로 선언해 에러를 수정할 수 있다.

```ts
declare module "png" {
  const src: string;
  export default src;
}
```

즉, declare 키워드는 타입스크립트가 알지 못하는 부분에 대해 컴파일러에 '이런 것이 존재한다'는 것을 알려준다.



**자바스크립트로 작성된 라이브러리**

tsconfig.json파일에서 any를 사용하지 못하게 했다면, 자바스크립트 라이브러리(타입이 any로 추론된다)를 사용했을 때 프로젝트가 빌드되지 않을 것이다.

이때 자바스크립트 내부 함수와 변수의 타입을 앰비언트 타입으로 선언하면 타입스크립트는 자동으로 .d.ts 확장자를 가진 파일을 검색하여 타입 검사를 진행해 문제 없이 컴파일 된다.



**타입스크립트로 작성된 라이브러리**

타입스크립트로 작성된 라이브러리도 자바스크립트 파일과 .d.ts파일로 배포된다.



**자바스크립트 어딘가에 전역 변수가 정의되어 있음을 타입스크립트에 알릴 때**

웹뷰를 개발할 때 네이티브 앱과의 통신을 위한 인터페이스를 네이티브 앱이 window 객체에 추가하는 경우가 많다. window에 변수나 함수를 추가하면 타입스크립트에서 직접 구현하지 않았더라도 실제 런타임 환경에서 해당 변수를 사용할 수 있다.



### 2. 앰비언트 타입 선언 시 주의점

**타입스크립트로 만드는 라이브러리에서는 불필요**

tsconfig.json의 declaration을 true로 설정하면 타입스크립트 컴파일러가 d.ts 파일을 자동으로 생성해주기 때문에 필요 없다.



**전역으로 타입을 정의하여 사용할 때 주의해야 할 점**

서로 다른 라이브러리에서 동일한 이름의 앰비언트 타입 선언을 한다면 충볼이 발생할 수 있다.



### 3.앰비언트 타입 선언을 잘못 사용했을 때의 문제점

.ts 파일 내의 앰비언트 변수 선언은 혼란을 야기할 수 있다. 앰비언트 변수 선언은 어느 곳에나 영향을 줄 수 있기 때문에 일반 타입 선언과 섞이게 되면 유지보수가 어려울 수 있다. 따라서 .d.ts 파일 내에서만 앰비언트 타입 선언을 하는 것이 권고된다.



### 4. 앰비언트 타입 활용하기

**타입을 정의하여 임포트 없이 전역으로 공유**

.d.ts 파일에서의 앰비언트 타입 선언은 전역 변수와 같은 역할을 수행한다. 따라서 모든 코드 내에서 import 없이 사용가능하다. 예를 들어 유틸리티 타입을 선언할 수 있다.



**declare type 활용하기**

커스텀 유틸리티 타입을 declare type으로 선언하여 전역에서 사용할 수 있다.



**declare module 활용하기**

CSS-in-JS라이브러리의 theme을 확장해 자동으로 완성되게 할 수 있다.



**declare namespace 활용하기**

.env 파일을 사용할 때, devlare namespace를 활용해 process.env로 설정 값을 쉽게 불러오고, 환경변수의 자동 완성 기능을 쓸 수 있다.



**declare global 활용하기**

전역 변수를 선언할 때 사용한다. 전역 변수인 Window 객체의 스코프에서 사용되는 모듈이나 변수를 추가할 수 있다.



### 5. declare와 번들러의 시너지

declare global로 전역 변수를 선언하는 것과 번드럴르 통해 데이터를 주입하는 절차를 함께 활용해 시너지를 낼 수 있다.

추가 예시 (from GPT)

1. **`declare global`로 전역 변수 선언**

`declare global`을 사용해 전역 변수를 TypeScript에서 안전하게 사용할 수 있도록 선언합니다.

```
ts


코드 복사
// src/types/global.d.ts
export {};

declare global {
  interface Window {
    __INITIAL_DATA__: Record<string, any>;
    __APP_ENV__: 'development' | 'production';
  }
}
```

2. **번들링 시 데이터 주입**

Webpack이나 Vite와 같은 번들러를 사용해 런타임 환경에서 전역 변수를 주입합니다.

**Webpack 예시:**

```
js


코드 복사
// webpack.config.js
const webpack = require('webpack');

module.exports = {
  // ...other config
  plugins: [
    new webpack.DefinePlugin({
      'window.__APP_ENV__': JSON.stringify(process.env.NODE_ENV), // 주입할 환경 변수
    }),
  ],
};
```

**Vite 예시:**

```
js


코드 복사
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  define: {
    'window.__APP_ENV__': JSON.stringify(process.env.NODE_ENV),
  },
});
```

3. **HTML에서 초기 데이터 주입**

초기 데이터를 서버 렌더링 또는 HTML 파일에 삽입해 브라우저에서 사용할 수 있도록 합니다.

**HTML 주입 예시:**

```
html


코드 복사
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Global Variables Example</title>
</head>
<body>
  <script>
    // 서버에서 초기 데이터를 주입
    window.__INITIAL_DATA__ = {
      user: { name: "John Doe", age: 30 },
      settings: { theme: "dark" }
    };
  </script>
  <div id="root"></div>
  <script src="/main.js"></script>
</body>
</html>
```

4. **React에서 전역 변수 사용**

주입된 데이터를 React 애플리케이션에서 사용할 수 있도록 불러옵니다.

```
tsx


코드 복사
// src/App.tsx
import React from 'react';

const App = () => {
  // 전역 변수 사용
  const initialData = window.__INITIAL_DATA__;
  const appEnv = window.__APP_ENV__;

  return (
    <div>
      <h1>Environment: {appEnv}</h1>
      <h2>User: {initialData.user.name}</h2>
      <p>Theme: {initialData.settings.theme}</p>
    </div>
  );
};

export default App;
```

5. **타입 검증**

`window.__INITIAL_DATA__`와 `window.__APP_ENV__`는 `declare global`로 정의된 타입을 참조하기 때문에, 개발 중에도 올바른 타입으로 안전하게 사용할 수 있습니다.

시너지 효과

1. **타입 안정성**: 전역 변수를 `declare global`로 선언하면, 런타임에서 주입된 값도 TypeScript의 타입 검사를 통해 안전하게 사용할 수 있습니다.
2. **유연한 데이터 전달**: 서버에서 초기 데이터를 HTML에 삽입하거나 번들러를 통해 런타임 환경 변수를 전달함으로써 다양한 환경에 적응 가능합니다.
3. **코드 간결성**: 데이터를 주입하는 절차와 전역 변수 접근을 결합해 복잡한 데이터 전달 계층을 단순화할 수 있습니다.



## 12.2 스크립트와 설정 파일 활용하기

### 1. 스크립트 활용하기

**실시간으로 타입을 검사하자**

실시간으로 에러 확인할 수 있는 스크립트: `yarn tsc -noEmit -incremental -w`

**타입 커버리지 확인하기**

`npx type-coverrage -detail`



### 2. 설정 파일 활용하기

**타입스크립트 컴파일 속도 높이기**

```ts
// tsconfig
{
  "compilerOptions": {
    ...
    incremental: true;
  }
}
```

스크립트: `yarn tsc -noEmit -incremental -diagnostic`



### 3. 에디터 활용하기

타입스크립트 서버 재실행 (VSCode): command + shift + P



## 12.3 타입스크립트 마이그레이션

### 1. 타입스크립트 마이그레이션의 필요성

상황에 따라 비즈니스 요구 사항의 변화를 반영할 수 있는 새로운 설계를 기반으로 타입을 작성하는 게 효율적일 수도 있다. 프로젝트의 규모와 특성 및 여건을 종합적으로 고려해보아야 한다.



### 2. 점진적인 마이그레이션

우선순위를 정해 시작하기를 권고하며, allowJs를 true, noImpicitAny를 false로 설정한 채 무기한으로 마이그레이션을 미루는 것은 지양해야 한다.



### 3. 마이그레이션 진행하기

1. 타입스크립트 개발 환경을 설정하고, 빌드 파이프라인에 타입스크립트 컴파일러를 통합한다.
   * 이 과정에서, tsconfig.json 파일 중 allowJs를 true, noImpicitAny를 false로 우선 지정해 오류를 발생하지 않게 한다.
2. 작성된 자바스크립트 파일을 타입스크립트 파일로 변환한다.
3. 기존 자바스크립트 파일을 모두 타입스크립트로 변환하는 작업이 완료 되었다면, allowJs를 false, noImpicitAny를 true로 설정하여 타입이 명시되지 않은 부분이 없는지 점검한다.



## 12. 4 모노레포

여러 프로젝트를 관리하는 경우 개별 프로젝트마다 별도의 레포지토리가 생성해 관리하는 경우가 일반적이다. 하지만 공통적인 코드가 많다면 여러 프로젝트를 하나의 레포에서 관리할 수도 있다.

### 1. 분산된 구조의 문제점

여러 프로젝트에 동일한 코드를 복붙한 경우, 버그를 찾았을 때 프로젝트의 개수만큼 반복적인 수정 작업을 수행해야 한다.



### 2. 통합할 수 있는 요소 찾기

모든 요소를 통합할 수 없는 환경이 대부분이기에, 공통으로 통합할 수 있는 요소를 찾아야 한다. 소스코드가 완전히 동일하지 않다면 통합을 위해 일부 수정해야 한다.



### 3. 공통 모듈화로 관리하기

소스코드를 수정한 후, 모듈화를 통해 통합할 수 있다. 단, 공통 모듈화를 통해 복붙의 번거로움을 줄일 수는 있지만 새로운 공통 모듈이 필요하다면 개발자는 새로운 레포지토리를 생성하고 개발환경 설정부터 테스트를 별도로 수행해야 한다.



### 4. 모노레포의 탄생

모노레포란 버전 관리 시스템에서 여러 프로젝트를 하나의 레포지토리로 통합하여 관리하는 소프트웨어 개발 전략이다. (기존 방식: 모놀리식)

**모노레포로 관리했을 때의 장점**

Lint, Ci/CD 등 개발 환경 설정도 통합적으로 관리해 불필요한 코드 중복을 줄여준다.

개별적으로 프로젝트를 형성하는 폴리레포와는 다르게 공통 모듈도 동일한 프로젝트 내에서 관리되므로 별도의 패키지 관리자를 통해 모듈을 게시하지 않아도 된다. (기능 변화를 쉽게 추적할 수 있다.)



**모노레포로 관리했을 때의 단점**

레포지토리가 거대해질 수 있다. 

하나의 레포지토리에 여러 팀의 이해관계가 얽혀있다면 소유권과 권한 관리가 복잡해질 수 있다.

