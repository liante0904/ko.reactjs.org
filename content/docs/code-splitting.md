---
id: code-splitting
title: Code Splitting
permalink: docs/code-splitting.html
---

## 번들링 {#bundling}

대부분 React 앱들은 [Webpack](https://webpack.js.org/) 이나 [Browserify](http://browserify.org/) 같은 툴을 사용하여 여러 파일을 하나로 병합한 "번들 된" 파일을 웹 페이지에 포함하여 한 번에 전체 앱을 로드 할 수 있습니다. 

#### 예시 {#example}

**App**

```js
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```

```js
// math.js
export function add(a, b) {
  return a + b;
}
```

**Bundle**

```js
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

> 주의
>
> 실제 번들은 위 예시와는 많이 다르게 보일 겁니다.

[Create React App](https://github.com/facebookincubator/create-react-app)이나 [Next.js](https://github.com/zeit/next.js/), [Gatsby](https://www.gatsbyjs.org/) 혹은 비슷한 툴을 사용한다면 여러분이 설치한 앱에서 Webpack을 같이 설치했을 겁니다.

이런 툴을 사용하지 않는다면 여러분이 스스로 번들링을 설정해야 합니다. 그런 경우, Webpack의 
[설치하기](https://webpack.js.org/guides/installation/) 문서와
[시작하기](https://webpack.js.org/guides/getting-started/) 문서를 참조해 주세요.

## Code Splitting {#code-splitting}

번들링은 훌륭하지만 여러분의 앱이 커지면 번들도 커집니다. 특히 큰 규모의 서드파티 라이브러리을 추가할 때 실수로 앱이 커져서 로드 시간이 길어지는 것을 방지하기 위해 코드를 주의 깊게 살펴야 합니다.

번들이 거대해지는 것을 방지하기 위한 좋은 해결방법은 번들을 "스플리팅" 하는 것입니다.
[Code-Splitting](https://webpack.js.org/guides/code-splitting/)은 런타임시 여러 번들을 동적으로 만들고 로드 하는 것으로  Webpack 와 Browserify ([factor-bundle](https://github.com/browserify/factor-bundle)) 같은 번들러들이 지원하는 기능입니다.


Code-splitting 은 여러분의 앱을 "lazy-load" 하게 도와주는데 이는 앱 사용자에게 획기적인 성능 향상을 하게 합니다.
앱의 코드 양을 줄이지 않고도 사용자가 필요로 하지 않는 코드를 로드하지 않게 하며 앱이 초기화 로드에 필요한 비용을 줄여줍니다.

## `import()` {#import}

 앱에 Code Splitting을 도입하는 가장 좋은 방법은 동적 `import()` 문법을  사용하는 방법입니다.

**Before**

```js
import { add } from './math';

console.log(add(16, 26));
```

**After**

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

> 주의
>
> 동적 `import()` 문법은 아직 ECMAScript (JavaScript)의 표준 문법이 아니라
> [프로포절](https://github.com/tc39/proposal-dynamic-import)입니다.
> 동적 `import()`은 가까운 미래에 표준에 추가 될 것으로 보입니다.

Webpack이 이 구문을 만나게 되면 앱은 자동으로 Code Splitting 하게 됩니다. 
만약 크리에이트 리액트 앱을 사용하고 있다면 이미 Webpack이 구성이 되어 있기 때문에 즉시 [사용](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#code-splitting) 할 수 있습니다.
[Next.js](https://github.com/zeit/next.js/#dynamic-import) 역시 지원 합니다.

만약 스스로 Webpack을 구성하고자 한다면 Webpack의 
[Code Splitting 가이드](https://webpack.js.org/guides/code-splitting/)를 참조 하세요. Webpack 설정은 [가이드](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)에 있습니다.

[Babel](http://babeljs.io/)을 사용할 때는 Babel이 동적 import를 할 수 있지만 변환하지는 않도록 합니다. 이를 위해 [babel-plugin-syntax-dynamic-import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import)를 사용 하세요.

## `React.lazy` {#reactlazy}

> 주의
>
> `React.lazy` 와 Suspense 는 아직 서버 사이드 렌더링 을 사용 할 수 없습니다. 만약 server rendered 된 앱에서 Code Splitting 을  사용하고자 한다면 [Loadable Components](https://github.com/smooth-code/loadable-components) 를 추천 합니다. 이것은 [server-side rendering 와 번들 스플리팅 가이드](https://github.com/smooth-code/loadable-components/blob/master/packages/server/README.md) 입니다.

`React.lazy` 함수를 사용하면 dynamic import를 사용해서 컴포넌트를 렌더링 할 수 있습니다.

**Before**

```js
import OtherComponent from './OtherComponent';

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```

**After**

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```
이 구성요소(MyComponent) 가 렌더링될 때 `OtherComponent` 를 포함한 번들을 자동으로 로드합니다.

`React.lazy`는 동적 import()를 호출하는 함수 형태로 사용됩니다. 이 경우 React 컴포넌트를 
export `default`로 해석되는 Promise로 리턴해야 합니다.  

### Suspense {#suspense}


`OtherComponent`를 포함하는 모듈이 `MyComponent`를 렌더링하기 전까지 로드가 완료되지 않은 경우 로드 상태 표시처럼 로드가 다 되기를 기다리는 동안 fallback content의 일부를 보여 주어야 하며 `Suspense` 컴포넌트를 사용하여 처리할 수 있습니다. 

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

`fallback` 기능은 컴포넌트가 로드가 될 때까지 기다리는 동안 렌더링하려는 모든 React 요소에 적용 가능합니다. `Suspense` 컴포넌트는 lazy 컴포넌트를 감쌉니다. 하나의 `Suspense` 컴포넌트로 여러 lazy 컴포넌트를 감쌀 수도 있습니다.


```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```

### Error boundaries {#error-boundaries}
네트워크 장애 같은 이유로 다른 모듈이 로드에 실패한 경우 에러를 발생시킬 수 있습니다. 이때 [Error Boundaries](/docs/error-boundaries.html)를 이용하여 사용자의 경험과 복구 관리를 처리할 수 있습니다.
Error Boundary를 만들고 lazy 컴포넌트를 감싸면 네트워크 장애가 발생했을 때 에러를 표시할 수 있습니다.

```js
import MyErrorBoundary from './MyErrorBoundary';
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

## Route-based code splitting {#route-based-code-splitting}

앱에서 Code Splitting을 도입할 위치를 결정하는 것은 조금 까다롭습니다. 
여러분은 번들을 균등하게 분배할 곳을 찾고 사용자의 경험을 헤치지 않기를 원합니다.

이를 시작하기 좋은 장소는 경로 입니다. 웹 페이지 로드 시간은 페이지 전환에 어느정도 발생하며 대부분 페이지를 한번에 렌더링하기 때문에
사용자가 페이지를 렌더링하는 동안 다른 요소와 상호작용하지 않습니다.

`React.lazy`를 [React Router](https://reacttraining.com/react-router/) 라이브러리를 사용해서 애플리케이션에 경로 기반 Code Splitting을 설정하는 예시입니다.

```js
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

## Named Exports {#named-exports}

`React.lazy` 는 현재 default exports만 지원 합니다. named exports를 사용하고자 한다면 default로 이름을 재정의한 중간 모듈을 생성 할 수 있습니다. 이렇게 하면 treeshaking이 계속 동작하고 사용하지 않는 컴포넌트는 가져오지 않습니다.

```js
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;
```

```js
// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";
```

```js
// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```
