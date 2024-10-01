# 프론트엔드 에러 핸들링

## 목차

1. 프론트엔드에서 에러란?
2. 예외(에러)를 분류해보자
3. 에러를 처리하는 방법
   1. 여러 에러 처리 방법
   2. 선언적으로 에러 처리하기(errorBoundary)
   3. 상황에 따라 어떻게 에러를 보여줄 수 있을까?
4. 에러 핸들링 설계하기

## 프론트엔드에서 에러란?

소프트웨어에서의 에러는 컴파일 에러, 런타임 에러로 나눌 수 있다.

컴파일 에러란 컴파일 시 컴파일러가 해석하지 못해서 발생하는 경우를 뜻한다. 예를 들어 잘못된 타입을 사용했을 때 나타날 수 있다. 이러한 컴파일 에러는 TypeScript를 사용하여 방지할 수 있다.

런타임 에러란 프로그램이 동작할 때 발견할 수 있는 에러이다. 존재하지 않는 DOM 요소에 접근하는 것을 예시로 들 수 있다.

컴파일 에러는 TypeScript, ESLint, Webpack 등의 도구를 이용해 캐치가 가능하다. 자바스크립트는 이러한 컴파일 에러가 아닌, 런타임 에러를 ‘예외’라고 부른다.

## 예외(에러)를 분류해보자.

![alt text](image.png)

웹 환경에서의 에러는 두 가지 기준으로 나눌 수 있다

1. 브라우저 환경에서- 예측 가능 vs 예측 불가능
2. 사용자가- 해결 가능 vs 해결 불가능

### 1. 예측 가능하며 사용자가 해결할 수 없는 에러

고의적으로 비정상적인 접근을 한 경우이다. 이는 보안과 관련되어있고 CORS, XSS 등을 고려해야 한다.

### 2. 예측 불가능하고 사용자가 해결할 수 없는 에러

`HTTP 상태 코드 500` 번대 오류로 서비스 장애, 특정 브라우저에서 나타나는 호환성 오류, 저사양 기기 사용 오류 등이 있다.

Sentry 등의 모니터링 도구를 활용하여 대응해야 하며, 사용자가 고객센터 문의 등을 활용할 수 있도록 해야한다.

### 3. 예측 불가능하며 사용자가 해결할 수 있는 에러

`네트워크` 오류가 이에 해당한다. 일시적인 문제임을 알리고, 재시도 가이드를 제공해야한다.

일시적인 에러로 모든 부분의 요청이 막히면 사용자에게 불편한 경험을 제공하게 되어서 에러의 영향 범위를 최소화 하는게 중요하다.

### 4. 예측 가능하며 사용자가 해결할 수 있는 에러

`HTTP 상태 코드가 명확한 에러`이다. 미리 정의되고 예상된 상황에서 발생한다.

인증 에러 ,잘못된 접근으로 인한 에러 등 400대 에러가 이에 해당한다.

각 상태 코드에 따른 적절한 가이드와 별도 에러 페이지를 제공해야한다.

예를 들어 인증 오류의 경우에는 사용자가 로그인을 시도할 수 있도록 유도해야 하며, 권한 오류의 경우에는 관리자에게 권한 요청을 하여 사용자 스스로 해결할 수 있어야 한다.

이렇게 에러의 예측, 해결 가능 여부에 따라 적절한 대응 방법으로 에러를 핸들링 할 수 있다.

### 추가 고려사항

- 에러는 최대한 작은 범위에서 catch해야 한다.
  화면에 여러 api 요청이 있을 때 하나의 요청에서 에러가 난다면, 다른 요청에는 영향을 미치지 말아야 한다. 따라서 요청이 일어나는 각 컴포넌트 마다 적절하게 에러를 핸들링 해 주어야 한다.
- HTTP Status를 기준으로 처리되는 에러에는 '합의된 에러'임을 나타내는 flag 값을 사용할 수 있다.
  이 중 `예측 가능하며 사용자가 해결할 수 있는 에러` 의 경우에는 백엔드와 합의하여 각 상태에 따른 에러코드를 정리해두는 게 좋다.
  예를 들어
  **400 Bad Request** 의 에러 코드를 받았을때를 생각해보자.
  지피티 사용하기
  400대 에러 상황 예시 적어주고, 400 대 에러 상태코드를 분리할 수 있는 상황으로,

## 에러를 처리하는 방법

1. try-catch문

```javascript
try {
  const result = await fetch("https://api.example.com/data");
  console.log(result);
} catch (error) {
  console.error("에러 발생:", error.message);
}
```

2. promise catch문

```javascript
fetch("https://api.example.com/data")
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error("네트워크 에러:", error));
```

3. axios interceptor

```javascript
import axios from "axios";

axios.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error("axios 에러:", error.message);
    return Promise.reject(error);
  }
);
```

4. react-query onError

```javascript
import { useQuery } from "react-query";

const { data, error } = useQuery("userData", fetchUserData, {
  onError: (error) => {
    console.error("데이터 fetching 에러:", error.message);
  },
});
```

5. ErrorBoundary

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error("에러 발생:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}

// 사용 예시
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>;
```

이러한 에러 처리 방식에는 \*명령형 에러-선언형 에러로 나눌 수 있다.

> \*명령형 프로그래밍은 프로그램의 상태를 변경하는 명령문을 순차적으로 나열하여 "어떻게" 동작해야 하는지 상세히 기술한다. 반면 선언형 프로그래밍은 원하는 결과를 선언하고, 그 결과를 얻기 위한 과정은 추상화하여 "무엇을" 해야 하는지에 초점을 맞춘다. 명령형은 세밀한 제어가 가능하지만 코드가 복잡해질 수 있고, 선언형은 가독성과 유지보수성이 높지만 때로는 성능 최적화가 어려울 수 있다.

- 명령형 에러 처리
  에러 처리 로직이 비즈니스 로직과 함께 섞여 있다.
  세밀한 에러 처리가 가능하고, 특정 상황에 따라 맞춤형으로 처리하기 쉽다는 장점이 있다.
  그러나 에러 상태와 성공 상태가 함께 있어서 코드의 가독성이 저하되고, 유지보수가 어려울 수 있다.
  위의 방법 중에는 errorBoundary 를 제외한 것들이 이에 속한다.(react query 의 onError 는 선언적이라고도 볼 수 있다.)

- 선언형 에러 처리(errorBoundary)
  에러 처리 로직을 컴포넌트로 분리하는 것.
  명령형과 다르게 코드의 가독성이 향상되고 에러처리 로직을 한곳에서 볼 수 있으며 컴포넌트의 재사용성이 증가한다는 장점이 있다.
  그러나 세밀한 에러 처리가 어려울 수 있다는 단점이 있다.

두 방식 모두 장단점이 있어 흑백 논리로 평가하기 보다는 장단점을 모두 이용할 수 있게 적절하게 사용하는게 중요하다.

### 상황에 따른 에러처리

에러 처리는 애플리케이션의 다양한 계층과 상황에 따라 다르게 접근할 수 있다.

GET 요청의 경우, 데이터 조회 실패 시 해당 섹션에 대체 성공-로딩-에러 상태에 따른 UI를 표시하는 것이 적절하다. 반면, POST 요청과 같은 사용자 액션의 결과는 페이지 전환보다는 toast 메시지를 통해 알리는 것이 사용자 경험 측면에서 유리할 수 있다.

로그인 프로세스를 예로 들어보자.

1. 최하위 레벨 (API 호출):

```javascript
async function loginUser(credentials) {
  try {
    const response = await fetch("/api/login", {
      method: "POST",
      body: JSON.stringify(credentials),
    });
    if (!response.ok) {
      throw new CustomError("로그인 실패", response.status);
    }
    return await response.json();
  } catch (error) {
    console.error("로그인 에러:", error);
    throw error instanceof CustomError
      ? error
      : new CustomError("네트워크 오류", 500);
  }
}
```

2. 중간 레벨 (react-query mutation):

```javascript
const loginMutation = useMutation(loginUser, {
  onError: (error) => {
    toast.error(`로그인 실패: ${error.message}`);
  },
  onSuccess: () => {
    navigate("/home");
  },
});
```

3. 상위 레벨 (컴포넌트 및 ErrorBoundary):

```jsx
function UserInfoSection() {
  const { data, error } = useQuery("userInfo", fetchUserInfo);

  if (error) throw error;

  return <div>{data.name}</div>;
}

function Home() {
  return (
    <ErrorBoundary fallback={<div>사용자 정보를 불러올 수 없습니다.</div>}>
      <UserInfoSection />
    </ErrorBoundary>
  );
}
```

이러한 구조에서, 로그인 시도 시 발생하는 에러는 최하위 레벨에서 커스텀 에러로 변환되어 전파된다. react-query의 mutation은 이 에러를 잡아 toast 메시지로 사용자에게 알린다. 로그인 성공 후 홈 화면의 사용자 정보 로딩 시 발생하는 에러는 ErrorBoundary에 의해 해당 섹션에 대체 UI를 표시한다.

이렇게 각 레벨, 보여주어야 하는 UI 등 상황에 따라 적절한 처리 방식을 고민해야 한다.

## 에러 핸들링 설계 및 실제 프로젝트에서 적용한 에러 핸들링

위의 내용들로 우리 프로젝트에서는 어떻게 에러를 핸들링 할 지 설계할 수 있다.

1. 에러 유형 분석 및 분류:

   - 예상 가능한 에러와 예상 불가능한 에러를 구분한다.
   - 사용자가 해결 가능한 에러와 불가능한 에러를 분류한다.
   - 각 유형별 대응 전략의 초안을 수립한다.

2. 백엔드와의 협업:

   - \*`에러 코드 체계`를 협의한다.
   - 각 에러 코드에 따른 상세 메시지를 정의한다.
   - API 응답 구조에 대해 합의한다. (예: { code: string, message: string })

> \*HTTP 상태 코드에 대한 상세 에러 메시지 정의의 예시
> 400 Bad Request: 4001: "필수 파라미터가 누락되었습니다.", 4002: "데이터 형식이 올바르지 않습니다."
> 401 Unauthorized: 4011: "인증 토큰이 유효하지 않습니다. 다시 로그인해주세요." , 4012: "인증 세션이 만료되었습니다. 재로그인이 필요합니다."
> 403 Forbidden: 4031: "접근이 거부되었습니다. 관리자에게 문의하세요." , 4032: "사용 한도를 초과하였습니다."

7. 최하위 레벨 에러 처리 전략:

   - API 클라이언트 레벨에서 에러를 포착하는 방식을 결정한다.
   - 커스텀 에러 클래스를 설계하고, 에러 변환 로직을 구현한다.
   - 네트워크 오류, 타임아웃 등 공통 에러에 대한 처리 방식을 정의한다.

8. 에러 표시 전략 수립:

   - HTTP 메소드에 따른 에러 표시 방식을 결정한다.
     - GET: ErrorBoundary를 활용한 에러 화면 표시
     - POST, PATCH, DELETE, PUT: Toast 메시지 활용
   - 각 에러 유형별 UI/UX 디자인을 기획한다.

9. 중간 레벨 에러 처리 구현:

   - react-query 또는 redux-toolkit 등 상태 관리 도구에서의 에러 처리 로직을 설계한다.
   - 전역 에러 핸들러와 개별 쿼리/뮤테이션 에러 핸들러의 역할을 정의한다.

10. 상위 레벨 에러 처리 구현:

    - ErrorBoundary 컴포넌트의 적용 범위를 결정한다. (전역, 페이지별, 섹션별)
    - Fallback UI의 디자인 및 구현 방식을 결정한다.

11. 사용자 피드백 시스템 구축:

    - Toast 컴포넌트 또는 라이브러리 선정
    - 에러 메시지 표시 규칙 수립 (지속 시간, 스타일 등)

12. 로깅 및 모니터링 전략:

    - 클라이언트 사이드 로깅 시스템 구축 방안을 수립한다.
    - Sentry 등 외부 모니터링 도구의 도입을 검토한다.

13. 테스트 전략 수립:

    - 각 에러 시나리오에 대한 단위 테스트 및 통합 테스트 계획을 수립한다.
    - 에러 핸들링 로직에 대한 테스트 코드 작성 방침을 정한다.

14. 문서화 및 팀 내 공유:

    - 에러 핸들링 가이드라인 문서를 작성한다.
    - 팀 내 코드 리뷰 및 지식 공유 세션을 계획한다.

15. 점진적 개선 계획:
    - 초기 구현 후 지속적인 모니터링 및 개선 계획을 수립한다.
    - 사용자 피드백을 수집하고 분석하는 프로세스를 구축한다.

이러한 플로우를 따르면서, 프로젝트의 특성과 팀의 역량을 고려하여 각 단계를 유연하게 조정할 수 있다. 중요한 것은 체계적인 접근을 통해 일관성 있고 효과적인 에러 핸들링 시스템을 구축하는 것이다.

> 실제 프로젝트에서 적용한 에러는 추후 추가...

---

레퍼런스

> https://happysisyphe.tistory.com/52

> https://velog.io/@baby_dev/공식에서의-에러핸들링-feat.-에러-바운더리

> [클라이언트의 사용자 중심 예외 처리](https://jbee.io/articles/react/%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8%EC%9D%98%20%EC%82%AC%EC%9A%A9%EC%9E%90%20%EC%A4%91%EC%8B%AC%20%EC%98%88%EC%99%B8%20%EC%B2%98%EB%A6%AC)

> https://comprogramming.tistory.com/121

> https://velog.io/@himprover/프론트엔드-에러-핸들링에-대한-고민
