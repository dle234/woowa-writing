# SSR 과 CSR 이해하기 

### 🗣️ SSR 과 CSR 의 차이점에 대해 알아보자!

SSR은 서버 사이드 렌더링, CSR은 클라이언트 사이드 렌더링이다.
글자 그대로 SSR은 서버에서 html을 만들고, CSR은 브라우저에서 html을 만든다. 하지만 여기에는 몇 가지 의문이 생길 수 있다.

> 💭 _ Nextjs 같은 프레임워크 없이 자바스크립트로 SSR을 구현하려면 어떻게 해야할까? nodejs 에서 html을 만드려면 어떻게 해야하지?_
> 💭 렌더링이 정확하게 뭘까? 서버에서 렌더링 되는 것과 클라이언트에서 렌더링 되는 방식에 어떤 차이가 있을까?
> 💭 여기서 말하는 렌더링과 브라우저 렌더링은 다른걸까?
> 💭 SSR에서 말하는 서버는 무엇일까? api 서버와는 다른걸까?

이러한 의문들을 해결하기 위해서는 웹의 발전 과정과 JavaScript 개발 환경에 대한 이해가 먼저 해결되어야 한다고 생각해서 간단하게 정리해보았다.

# <1> 웹 개발의 역사적 타임라인

## 1. 초기 웹 (1990년대)

초기의 웹은 단순한 정적 HTML 파일을 제공하는 `순수한 SSR 형태`였다.

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="./main.js"></script>
    <script src="./foo.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

### 주요 문제점

- **전역 스코프 오염**: 각 모듈에 독립적인 스코프가 생성되지 않아 자바스크립트 파일들이 서로 영향을 주었다.
- **의존성 관리 문제**: 모듈 간 로딩 순서가 보장되지 않았다.
  예를 들어

```
<script src="jquery-plugin.js"></script>   <!-- jQuery가 없어서 에러 발생 -->
<script src="jquery.js"></script>

```

이 경우 의존성을 명시적으로 선언할 방법이 없어서 개발자가 수동으로 순서를 맞춰야 했다.

- **성능 저하**: 순차적 로딩으로 인한 성능 문제. 자바스크립트를 모두 로딩하는 동안 아무 동작도 하지 못했다.

## 2. 초기 해결 패턴 (2000년대 초반)

위와 같은 문제점을 해결하기 위해 다음과 같은 패턴들이 등장했다.

```jsx
// IIFE 패턴
var Module = (function () {
  var private = "private"; // 비공개 변수
  return {
    getPrivate: function () {
      // 공개 메서드
      return private;
    },
  };
})();

// 네임스페이스 패턴
var MyApp = MyApp || {};
MyApp.Module = {
  // 모듈 내용
};
```

이를 통해 전역 스코프 오염을 해결하고 , 모듈 단위로 구조화가 가능해졌다.
하지만 여전히 다음과 같은 한계점이 존재했다.

- 전역 객체(MyApp, Module) 생성 필요
- 수동적인 의존성 관리
- 모듈 재사용의 어려움
- 파일 단위 스코프 부재

## 3. AJAX의 등장 (2005)

자바스크립트의 중요성이 증가하면서 Ajax가 등장했다.
이를 통해 기본 페이지는 서버에서 생성하되(SSR) 비동기 데이터 통신이 필요한 부분은 ajax 를 통해 부분 업데이트가 가능해졌다.(CSR)
또한 JSON 기반 데이터 통신의 기초를 제공해 주었다.

```html
// 기본 페이지는 SSR로 제공
<!DOCTYPE html>
<html>
  <body>
    ... 생략
    <!-- 새 게시글 작성 폼 -->
    <form id="postForm">
      <input type="text" id="content" />
      <button type="submit">작성</button>
    </form>

    <script>
              // AJAX를 통한 부분 업데이트
              document.getElementById('postForm').onsubmit = function(e) {
                  e.preventDefault();

                  var xhr = new XMLHttpRequest();
                  xhr.open('POST', '/api/posts');
      ...생략

           // DOM 부분 업데이트
                  var postsDiv = document.getElementById('posts');
                  postsDiv.innerHTML += `
                              <div class="post">${newPost.content}</div>
                          `;
                      }
                  };
      ... 생략
    </script>
  </body>
</html>
```

## 4. V8 엔진 등장 (2008)

구글의 크롬이 부상하면서 구글에서 크롬 브라우저 용으로 v8 자바스크립트 엔진을 개발했다.
기존의 자바스크립트 엔진보다 빠르고 서버로 사용할 만큼 좋은 성능을 가졌다.

### 특징

- JIT(Just-In-Time) 컴파일 : V8이 실행 시점에 기계어로 컴파일. 인터프리터 방식보다 훨씬 빠른 실행 속도를 가지게 되었다.
- 효율적인 메모리 관리

> 왜 인터프리터보다 JIT 컴파일 방식이 더 빠를까?
- 인터프리터 : 매번 한 줄씩 해석하고 실행한다.
- JIT 컴파일 : 실행 시점에 필요한 코드를 기계어로 컴파일하고, 컴파일 된 코드를 캐싱하여 재사용한다. + 컴파일 과정에서 코드 최적화가 가능하다.

## 5. Node.js 등장 (2009)

nodejs 는 V8 엔진 기반의 서버사이드 자바스크립트 실행 환경이다.
node js는 자바스크립트를 서버로 활용하기 위한 도구가 될 수 있다.
기존의 자바스크립트는 웹 브라우저 위에서만 실행할 수 있었는데, 자바스크립트 런타임인 nodejs 가 등장한 이후 다른 환경에서도 자바스크립트를 실행할 수 있게 되었다.

- **브라우저 환경과 nodejs 서버 환경 비교하기**

> [브라우저 환경]
> Chrome Browser
> ├── V8 엔진 (JS 실행)
> ├── Web APIs (DOM, AJAX, Timer 등)
> └── Event Loop

> [Node.js 환경]
> Node.js Runtime
> ├── V8 엔진 (JS 실행)
> ├── libuv (비동기 I/O)
> └── Node.js APIs (fs, http 등)

## 6. 모듈 시스템의 발전

### CommonJS (2009)

CommonJS 는 기존의 nodejs 문법에 모듈화가 추가된 모듈 시스템이다.
Node.js의 등장으로 자바스크립트를 범용적으로 쓰기 위해 표준화된 모듈 시스템이 필요해졌다. commonjs 는 이를 위해 등장했으며 다음과 같은 역할을 했다.

- 표준화된 모듈 시스템(require, export 로 독립적인 스코프 제공)
- 공통 패키지 모듈 저장소 (이후 npm으로 발전)

```jsx
const module = require("./module");
module.exports = function () {};
```

그러나 브라우저에서 해당 문법을 이해하지 못해서 별도의 번들러(webpack, browserify 등)가 필요했다.

### AMD (2009)

CommonJS의 브라우저 한계를 해결하기 위해 등장했다.

```jsx
define(["dependency"], function (dependency) {
  return {
    method: function () {},
  };
});
```

AMD 는 다음과 같은 특징이 있다.

- 브라우저 환경에 최적화
- 비동기 모듈 로딩 지원
- 의존성 명시적 선언
- RequireJS 같은 라이브러리를 통해 사용

그러나 문법이 복잡하고, 콜백 중첩이 발생하며 브라우저만을 위한 솔루션이었다.

## 7. Webpack 등장(2012)

![alt text](image-1.png)

여러 모듈 시스템이 나왔지만, 이러한 모듈 시스템을 잘 사용하기 어려웠다. 여기에는 다음과 같은 문제점이 있었다.

- 브라우저에서는 모듈들을 불러올 때 각각 네트워크 요청을 해줘야 한다.
- CommonJS, AMD, UMD 등 서로 다른 모듈 시스템 문법.

이러한 모듈 시스템을 잘 사용하기 위해 모듈 번들러들이 등장했다.

웹팩은 모듈 번들러 중 하나로 모든 파일을 모듈로 관리한다.

```
// 예시
import sum from './math';
import './style.css';
import logo from './logo.png';

```

웹팩은 4가지 주요 기능이 있다

1. entry-output : import/require 로 연결된 모든 파일 수집하며 의존성 그래프를 그린다.
2. loader : 모든 파일들을 js에서 다룰 수 있는 형태로 만든다.(즉, 브라우저에서 이해할 수 있는 코드로 만든다.)
   ex) 이미지 → 파일 경로 문자열 또는 Base64 문자열 (file-loader)
   CSS → DOM에 스타일을 주입하는 JavaScript 코드 (css-loader)
   ES6 → ES5 (babel-loader)
3. 번들링
4. plugin : 번들된 결과물을 처리,최적화 한다. js를 난독화 하거나 css 를 추출하는 등의 기능을 할 수 있다.

이 과정을 거치며 babel, polyfill 등의 기능 또한 함께 제공되는데, 자세히 알아보자

### polyfill

폴리필은 es6 를 es5 이전 문법으로 변환할 때 없는 메서드나 객체를 직접 구현해주는 역할을 한다.
즉, 기능 자체를 추가해주기 때문에 폴리필이 없다면 참조 에러(ReferenceError)를 마주칠 수 있다.

```
// 원본 코드
const arr = [1, 2, 3];
arr.includes(2);
new Promise();

// Polyfill 추가 후
if (!Array.prototype.includes) {
    Array.prototype.includes = function(item) {
        return this.indexOf(item) !== -1;
    }
}
if (!window.Promise) {
    window.Promise = function() {
        // Promise 구현
    }
}
// ✅ 이제 includes와 Promise 사용 가능

```

### Babel (2014)

바벨은 es6 를 es5로 변환시켜주는 트랜스파일러이다.
새로운 문법을 옛날 문법으로 변환시켜주며 다음과 같은 기능 또한 지원해준다.

- JSX 트랜스파일링 지원
- TypeScript 지원: 타입스크립트를 자바스크립트로 변환
  단, 타입 체크가 안되기 때문에 컴파일 시 타입체크를 하고 싶다면 TypeScript 컴파일러를 사용하거나 ts-loader 를 사용해야한다.

만약 바벨이 없다면 자바스크립트가 구문을 제대로 이해하지 못해 syntax error가 날 수도 있다.

```
// 원본 코드 (ES6+)
const arrowFn = () => {};
class MyClass {}

// Babel 변환 후 (ES5)
var arrowFn = function() {};
function MyClass() {}

```

## 8. ES6(ES2015) 발표 (2015)

ESM은 특정 환경(브라우저/서버)만을 위한 솔루션이 아닌, 자바스크립트의 통합된 모듈 시스템이다.

```jsx
import { function } from './module';
export const newFunction = () => {};

```

기존 commonjs 만 지원해주던 라이브러리들이 esm 을 지원하기 시작했다.
ex) typescript, nextjs, jest
추가로, 최근 등장한 오픈소스 중 commonjs 지원을 없앤 경우도 존재한다.
ex) vite

> commonJs vs ESM

- esm 은 트리쉐이킹 가능, cjs 는 불가. cjs 는 동적 import 로 빌드 타임에 어떤 값이 불러와서 사용해질 수 있는지 가늠하기 어렵다.(cjs 자체가 트리쉐이킹이 불가한게 아니라 동적 import 가 불가한 것.)
- esm 은 cjs 보다 느림. cjs 는 동기적으로 실행되나, esm 은 전체 모듈 그래프가 구문 분석된 후 다음 코드가 평가되어 느리다.
- cjs 는 조건에 따른 require 이 가능해서 동적 로딩, 지연 로드 등에 유용하다.

> node package manager(npm)
> npm 을 통해 모듈을 배포가 가능하고, 외부의 모듈을 설치할 수 있다. 초기에는 nodejs 모듈에 한정되었지만, 최근에는 자바스크립트 패키지 매니저로 확장됐다.
> *package.json*은 설치한 모듈들의 메타데이터와 의존성을 정의하는 파일이다. 기본적으로 commonjs 문법을 사용하지만, 이 파일에 "type": "module" 설정을 추가해주면 esm을 기본 모듈시스템으로 사용 가능해진다.

## 9. 마무리

마지막으로 렌더링 환경을 정리해보자

![alt text](image-2.png)

사실 이 정보만으로는 위의 의문에 대답할 수 없다.
하지만,
CSR(여기서는 리액트)이 보낸 bundle.js 를 `브라우저에서 렌더링`하여 html을 만든다! 의 의미와,
SSR이 `서버(여기서는 NodeJS)에서 렌더링` 하여 html을 만든다! 가 무슨 의미인지는 알 것이다.

즉

> 리액트(CSR) -> ts,jsx,es6 ...웹팩으로 번들링(es5이전 버전으로 트랜스파일링, 최적화) -> 브라우저가 bundle.js 받고 DOM,CSSOM tree 그린 후 자바스크립트 엔진으로 자바스크립트를 실행함.(브라우저 렌더링)
> //사실 v8은 es6도 지원되나, 구형브라우저를 위해 트랜스파일링

> SSR(nodejs를 이용한 순수 SSR) -> nodejs 에서 자바스크립트를 실행하여 html 생성 -> 브라우저로 보냄 -> 브라우저에서 DOM+CSSROM tree 그림 (브라우저 렌더링)

앞으로 나올 CSR, SSR을 이해하기 위한 바탕이 완성됐다

# <2> SSR, CSR 코드 살펴보고 비교하기
그렇다면 예시를 보며 비교해보자
우리는 api 서버에서 영화 목록을 불러오고, 불러온 영화목록을 화면에 보여줘야하는 상황이다. 이때 뼈대 html 코드는 다음과 같다.

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="../assets/styles/reset.css" />
...생략
    <script src="./"></script>
    <title>영화 리뷰</title>
  </head>
  <body>
    <div id="wrap">
      <main>
          <section>
            <h2>지금 인기 있는 영화</h2>
            <ul class="thumbnail-list">
              <!--${MOVIE_ITEMS_PLACEHOLDER}-->
            </ul>
          </section>
        </main>
 ...생략
```

다음과 같은 html이 있을때 데이터를 fetch해서 최종 html 을 어떻게 완성하는지
SSR, SSR+CSR 별로 살펴보자.


## 순수한 SSR
🙎 가정 : 초기 페이지에 접속하면 영화 목록을 확인하고 싶어요
이때, 초기 렌더링 속도를 줄이기 위해 SSR 를 활용하고 싶어요.

### ⭐️ 렌더링을 위한 nodejs 코드
```js
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const router = Router();

const renderMoviePage = async (req, res) => {
  try {
    const templatePath = path.join(__dirname, "../../views", "index.html");

    const movies = await fetchMovies(); 
   // 1️⃣ 영화 목록 fetch
    const featuredMovie = movies[0];
    const moviesHTML = renderMovieItems(movies);
    let template = fs.readFileSync(templatePath, "utf-8");
    const renderedHTML = template
      .replace("<!--${MOVIE_ITEMS_PLACEHOLDER}-->", moviesHTML);
	// 2️⃣ html 생성
    res.send(renderedHTML);
  } catch (error) {
    console.error(error.message);
    res.status(500).send("Internal Server Error");
  }
};

export default router;
```
여기서는 ssr 서버에서 어떻게 렌더링하는지 알 수 있다.
- html 가져오기
- 영화 목록 fetch 해오기
- renderMovieItems(정적인 html파일을 return 해주는 자바스크립트 함수) 함수에 fetch한 데이터를 넣어 영화 목록 html을 생성한다.
- `<!--${MOVIE_ITEMS_PLACEHOLDER}-->` 부분을 위에서 생성한 html으로 바꿔준다. 
- 특정 라우팅 경로에서 다음과 같이 완성된 html을 보내준다.

## SSR+CSR
🙎 가정 : 초기 페이지에 접속하면 영화 목록을 확인하고, 영화를 클릭하면 영화의 상세 정보를 모달로 확인하고 싶어요.

이때, 초기 렌더링 속도를 줄이기 위해 SSR 를 활용하나, 그 이후부터 CSR로 동작했으면 좋겠어요.


### 폴더 구조
```
📦src
 ┣ 📂apis
 ┃ ┗ 📜fetchMovies.js
 ┣ 📂client
 ┃ ┣ 📂components
 ┃ ┣ 📂pages
 ┃ ┣ 📜App.jsx
 ┃ ┗ 📜main.js
 ┣ 📂server
 ┃ ┣ 📂routes
 ┃ ┃ ┗ 📜index.js
 ┗ ┗ 📜main.js
 
 client 와 server 폴더를 분리해주었다.
```

## client
client 에서는 리액트에서 한 것 처럼 구현해주면 된다.
이 부분은 첫 렌더링 후 동적인 웹사이트를 구성하기 위해 사용된다.

### main
```jsx
const initialData = window.__INITIAL_DATA__;

hydrateRoot(
  document.getElementById("root"),
  <BrowserRouter>
    <App movies={initialData.movies} movieDetail={initialData.movieDetail} />
  </BrowserRouter>
);
```
기존에 사용하던 리액트와 다르게 `initialData` 가 있고, createRoot 대신 `hydrateRoot`가 있는 것을 확인할 수 있다.

### App
```jsx
function App({ movies, movieDetail }) {
  return (
    <Routes>
      <Route path="/" element={<Home movies={movies} />} />
      <Route
        path="/detail/:id"
        element={<MovieDetail movies={movies} movieDetail={movieDetail} />}
      />
    </Routes>
  );
}
```
app 에서 다음과 같이 라우팅해주었다.

## server
```js
import { StaticRouter } from "react-router-dom/server";

const router = Router();
function getInitialDataScript(movies, movieDetail) {
  return `
    <script>
      window.__INITIAL_DATA__ = {
        movies: ${JSON.stringify(movies)},
        movieDetail: ${JSON.stringify(movieDetail)}
      };
    </script>
  `;
}

async function renderApp(location, res, movies, movieDetail) {
  try {
    const renderedApp = renderToString(
      <StaticRouter location={location}>
//1️⃣ StaticRouter 를 활용하면 서버에서도 클라이언트의 라우팅 구조를 그대로 사용할 수 있다
        <App movies={movies} movieDetail={movieDetail} />
      </StaticRouter>
    );

//...생략
}

router.get("/", async (req, res) => {
    const movies = (await fetchNowPlayingMovieItems()) || [];
    await renderApp("/", res, movies, null);
});
// 2️⃣ 영화 목록을 fetch 후 라우팅에 알맞게 html을 보내준다.

router.get("/detail/:id", async (req, res) => {
//...생략
});
// 3️⃣ 영화 상세 목록도 마찬가지로
export default router;

```
서버 코드이다. client 에서 사용한 코드를 정적인 html 코드로 바꾼 후 
각 라우트 마다 넣어주었다. `StaticRouter` 를 활용하면 경로에 따른 App rendering 이 가능하다.

이 코드는 어떻게 동작할까?

첫 접속 시 서버에서 렌더링된 html 파일이 전달되고(SSR), 그 이후 인터렉션에서는 CSR 이 작동할것이다. 

- 첫 접속 시 네트워크창
![](https://velog.velcdn.com/images/kky1373/post/b127a946-625d-4913-859c-0f4804bdcd9e/image.png)

- 클릭 시 네트워크창
![](https://velog.velcdn.com/images/kky1373/post/de802095-8c6d-4014-8e33-11ca06cb2a2c/image.png)


>  첫 접속하면 서버에서 HTML을 전달하고, 영화의 디테일을 보려고 클릭 시 click 이벤트와 navigate가 실행되어 URL이 변경되고 영화의 디테일을 보여줄 것이다. 그리고, 그 상태에서 새로고침 시 첫 접속이기 때문에 디테일 페이지가 서버에서 생성되어 완성된 HTML을 전달할 것이다.



정리하면 다음과 같다.

1. **초기 로드 (SSR)**
- 서버에서 HTML 생성하여 전달한다.
- webpack/server : JSX 트랜스파일링,경로 별칭(alias) 해결, asset 등을 처리해야 한다.

2. **스크립트 로드**
- HTML에 `<script defer src="/static/bundle.js"></script>` 포함
- 이는 webpack/client 설정을 통해 자동 주입된다.
- 스크립트가 로드되면 하이드레이션이 시작된다.

3. **하이드레이션 준비**
- window.__INITIAL_DATA__를 통해 서버 데이터를 주입한다.
- 이 데이터로 클라이언트도 서버와 동일한 상태 유지가 가능하다.

4. **하이드레이션 실행**
- 리액트가 브라우저의 기존 DOM과 INITIAL_DATA가 넣어진 상태로 생성된 가상 DOM을 비교하며, 일치하면 이벤트 리스너를 붙인다. 만약 일치하지 않는다면 에러가 발생한다.

5. **CSR 전환**
- 이후 모든 인터랙션은 클라이언트에서 처리한다.
- React가 일반적인 SPA처럼 동작한다.

## CSR vs SSR 장단점
코드로 SSR 을 알아보고, CSR 과 SSR 을 함께 사용하는 방법, 그리고 이 둘의 렌더링 방식이 어떻게 다른지 알아보았다.

마지막으로 이 두 방식을 비교해보자.
![](https://velog.velcdn.com/images/kky1373/post/49985b0f-2deb-49c0-8e0f-6be6fdfd5457/image.png)

```
단, 위 예시는 react 환경에서 CSR,SSR을 할 경우에 대한 예시이다.
```

### CSR이 유리한 경우:

- **사용자의 인터렉션이 많은 웹**
_`예: SNS, 웹 메일, 관리자 대시보드`_
 CSR은 클라이언트에서 자바스크립트를 실행하기 때문에 `즉각적인 UI 업데이트` 가능하다

- **서버 비용을 줄여야 할 경우**
정적 파일 제공만 하면 되므로 `서버 부하가 감소`한다.
추가로, 정적 파일만으로 구성되어 있어 CDN 배포가 용이하다.

### SSR이 유리한 경우:
- **초기 렌더링 속도가 중요한 경우**
_`예: 뉴스, 쇼핑몰 상품 페이지`_
SSR은 완성된 HTML을 바로 받아볼 수 있어 첫 렌더링 속도가 CSR보다 빠르다.
CSR은 서버로부터 js를(bundle.js) 초기에 모두 다운받아야 하고, 실행까지 완료되어야 하기 때문에 느리다.

> **💭 (+) 왜 CSR에서 fetch 하는 것 보다 SSR에서 fetch 하는게 비교적 빠른 경우가 많을까?**
> - **물리적 거리**
서버 ↔ API 서버: 보통 같은 데이터센터 내에 위치
브라우저 ↔ API 서버: 사용자의 위치에 따라 물리적 거리가 멀 수 있다.
> - **네트워크 환경**
서버: 안정적인 고속 네트워크 환경
브라우저: 사용자의 네트워크 상태에 따라 불안정할 수 있다 (3G, 4G, WiFi 등)

- **SEO 최적화가 중요한 경우**
_`예: 콘텐츠 중심 웹사이트, 마케팅 페이지`_
SSR 의 경우 검색 엔진이 완성된 HTML을 크롤링할 수 있어 SEO에 더 유리하다.
CSR의 경우에는 크롤링 할 때 초기 html을 크롤링 하기 때문에 불리하다. 
요즘은 Google과 같은 주요 검색엔진이 JavaScript를 실행하여 크롤링할 수 있지만, 모든 검색엔진이 그런 것은 아니며, JavaScript 크롤링은 HTML 크롤링보다 더 많은 리소스와 시간이 필요하다.

요즘은 SSR과 CSR을 함께 사용하는 하이브리드 방식이 많이 활용되고 있다.
Next.js와 같은 프레임워크들은 이러한 렌더링 방식들을 쉽게 구현할 수 있도록 도와주고 있다.

렌더링 방식에는 SSR, CSR만 있는 것이 아니라 SSG(Static Site Generation), ISR(Incremental Static Regeneration) 등 다양한 렌더링 전략들이 있다.

각 페이지나 컴포넌트의 특성을 잘 파악하고, 거기에 맞는 최적의 렌더링 방식을 선택해서 조합하자.


---

레퍼런스

- [모듈 시스템](https://medium.com/@hong009319/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-%ED%91%9C%EC%A4%80-%EC%A0%95%EC%9D%98-commonjs-vs-es-modules-306e5f0a74b1)
- [nodejs](https://urmaru.com/8)
- [v8엔진](https://ui.toast.com/weekly-pick/ko_20200228)
- [웹팩](https://ehddnjs8989.medium.com/webpack%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B4%85%EC%8B%9C%EB%8B%A4-c953181e79ad)
- [nodejs 와 브라우저 event loop 차이](https://dev.to/jasmin/difference-between-the-event-loop-in-browser-and-node-js-1113)

- [SSR,CSR 장.단점 1](https://velog.io/@yukimiau/SSR%EC%99%80-Next.js-%EA%B7%B8%EB%A6%AC%EA%B3%A0-SEO)

- [SSR,CSR 장.단점 2](https://yozm.wishket.com/magazine/detail/2330/)

- [SSR,CSR 코드 - 우테코 미션 깃허브](https://github.com/dle234/react-ssr)
