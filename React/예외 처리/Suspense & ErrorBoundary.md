## React.Suspense

1. 컴포넌트를 렌더링 할 준비가 안되었을 경우 
2. API 응답을 기다리는 경우 

### 컴포넌트를 렌더링 할 준비가 안되었을 경우

suspense 아래에 아직 render될 준비가 안된 컴포넌트가 있다면 loading UI(예비 컨텐츠)를 띄어줄 수 있다. 

```jsx
// <OtherComponent /> 컴포넌트는 동적으로 불러온다.
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // OtherComponent load될 때까지 <Spinner />를 보여준다. 
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```

특징 

- 하나의 suspense 컴포넌트로 여러 lazy 컴포넌트를 감쌀 수 있다.

### API 응답을 기다리는 경우

React Query를 사용할 때 API 응답이 도착하기 전까지 fallback을 띄울 수 있다. 

```jsx
// 🥕 Suspense가 없다면 ? 

return (
	{isLoading && <Loading />} 
	// 로딩 상태에 따라 로딩 컴포넌트를 렌더링하는 코드를 각 컴포넌트에 작성해야 한다. 
	{data && <Data />}
)

// 🥕 Suspense를 사용한다면 ?

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      suspense: true,
    },
  },
});

<Suspense fallback={<Loading />}>
		... // 아래의 모든 컴포넌트에서 API 응답을 기다리고 있을 때 fallback UI가 보여진다. 
</Suspense>
```

### 장점

- 로딩 컴포넌트를 중복해서 작성할 필요가 없다.
- 사용자에게 Loading 중임을 명시적으로 보여줄 수 있다.

### React Query에서 Suspense 옵션을 사용할 때 주의할 점 (with React Query)

> suspense 옵션이 true일 경우 onSuccess에서 setState를 하면 안됩니다.
> 

```jsx
// TComponent.tsx

useQuery(..., {
	onSuccess: (data) => {
		setState(data)
	},
	suspense: true
})
```

```jsx
<Suspense fallback={<Loading />}>
	<TComponent />
</Suspense>
```

<img width="980" alt="스크린샷 2022-11-08 오후 1 40 04 (1)" src="https://user-images.githubusercontent.com/52737532/200514618-c69acb14-6d2f-4178-8f15-fe5245e66ec2.png">

1. TComponent의 useQuery가 동작하여 API call를 한다.
2. TComponent가 unmount되고 fallback UI가 mount된다. 
3. API call이 success되었을 때 onSuccess가 동작한다. 여기서 setState를 실행하여 state를 변경하려 환다. 
4. TComponent는 unmount 상태기 때문에 setState를 해도 아무일도 발생하지 않는다. → re-render가 되지 않은 것처럼 보여진다. 

## ErrorBoundary

1. 여러 컴포넌트에서 발생할 수 있는 공통 에러를 상단에서 잡아 처리하고 싶은 경우 
2. API 에러 응답에 대한 처리를 공통적으로 하고 싶은 경우 

ErrorBoundary가 잡지 못하는 에러 

- 이벤트 핸들러에서 발생한 오류는 잡지 못한다.
- 비동기적 코드에서 발생한 오류는 잡지 못한다.
- 서버 사이드 렌더링 에러

**정리하자면 렌더링 중에 발생하는 오류가 아니면 잡지 못한다.** 
이벤트 핸들러 내에서 에러를 잡아야할 경우 try/catch를 사용할 것 

### 구현

모듈로 react에서 제공하고 이런 것이 아니다. 
[[공식 문서를 보며 구현](https://ko.reactjs.org/docs/error-boundaries.html#introducing-error-boundaries)](https://ko.reactjs.org/docs/error-boundaries.html#introducing-error-boundaries)하면 된다. 
ErrorBoundary가 작동할 수 있는 이유는 2가지의 생명주기 메서드 덕이다. 

1. `[static getDerivedStateFromError()](https://ko.reactjs.org/docs/react-component.html#static-getderivedstatefromerror)`
2. `[componentDidCatch()](https://ko.reactjs.org/docs/react-component.html#componentdidcatch)`

**getDerivedStateFromError**
- 자손 컴포넌트에서 오류가 발생됐을 때 호출된다.
- 오류를 매개변수로 받는다.
- 갱신된 state값을 반환해야한다.
- render 단계에서 호출되므로, 부수 효과를 발생시키면 안된다.

**componentDidCatch**
- 자식 컴포넌트에서 오류가 발생됐을 때 호출된다.
- 에러를 이용해서 side effect를 발생시키고 싶을 때 사용하면 된다. (commit 단계에서 호출)
    - 오류 로그 기록등을 위해 사용
- 2개의 매개변수를 전달 받는다. error, info(componentStack을 가지는 객체)

생명주기 메서드를 활용해야 하기 때문에 클래스 컴포넌트로 구현해야 한다. 
만약 hook을 사용하고 싶다면 HOC를 활용해보자. 

### 공통 에러를 상단에서 잡아 처리하고 싶은 경우

- 500 internal server error
    - 어느 컴포넌트에서나 API 요청을 한다면 발생할 수 있다.
- 네트워크 에러 (API 통신 중 네트워크 상태에 문제가 생겼다면)
- JS 런타임 에러 (개발자의 실수 undefined를 함수로 호출한다면)
- 401 에러같이 인증이 필요한 부분을 통합해서 관리하고 싶을 때

### API 에러 응답을 핸들링할 때 (with React Query)

우선 비동기 작업중 발생한 오류는 에러 바운더리가 잡을 수 없다.
하지만 react query 의 useErrorBoundary 옵션을 키면 react query가 내부적으로 API 통신 오류를 다음 렌더링 주기에 ErrorBoundary로 던져준다. 

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      useErrorBoundary: true, 
  },
});
```

400, 404 같은 에러는 각 컴포넌트에서 처리하고 있고, 500에러 이상부터 처리해주고 싶다면 

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      useErrorBoundary: error => error.response?.status! >= 500, 
  },
});
```

bool을 반환하는 함수를 넣어줘도 된다. 

### ErrorBoundary 재사용하기

ErrorBoundary의 props를 넘겨줘서 fallback UI를 주입시켜줄 수 있다. 
ErrorBoundary를 재사용하게 만드는 것의 장점은 부분적으로 fallback을 띄울 수 있다는 점이다. 
(전체 화면에 A, B 파트가 있고, A 파트에서 오류가 발생했을 때 B 파트는 정상적으로 작동하도록 할 수 있다.)

```jsx
// 개인적인 방식 

render() {
  if (this.state.hasError) {
    return this.props.fallback(this.state.error, this.reset);
  } // fallback 컴포넌트에 error 정보를 넘겨주기 위해서 함수로 props를 받음 

  return this.props.children; 
}

// 사용처

<ErrorBoundary
  fallback={(error: Error | AxiosError | null, errorBoundaryReset: () => void) => (
    <CommonError error={error} errorBoundaryReset={errorBoundaryReset} />
  )}
>
```

### fallback에서 벗어날 수 있게 하기 (with React Query)

fallback UI는 뒤로가기로 벗어날 수 없다. 
JS 문법적 오류는 사용자가 해결할 수 있는 부분이 없기 때문에 이를 벗어나는 것이 불가능하지만, 
API 에러같은 경우는 재시도로 벗어나게 할 수 있다. 

1. fallback UI를 벗어나기 위해서는 ErrorBoundary를 re-render 시켜야 한다. (props로 받은 children을 렌더링해야 하기 때문에) 
2. 만약 API 통신에서 에러가 날 경우 query가 inactive된다. 이를 다시 fetching 상태로 돌려야 API 요청을 재시도한다. 

공식 홈페이지에서는 `QueryErrorResetBoundary`와 react-error-boundary에서 지원해주는 `ErrorBoundary`를 사용하라고 한다. 하지만 react-error-boundary는 추가로 설치해야되는 라이브러리다. 
또, React에서 공식 문서에 작성되어 있는 ErrorBoundary 코드로는 reset이 동작하지 않는다. 
1과 2를 해결하기 위해 `setState`와 `QueryClient.refetchQueries`를 활용할 수 있다. 

```jsx
// ErrorBoundary를 re-render 시키자! 

reset = () => {
    this.setState(initialState);
};

// query를 refetch 하자!

queryClient.refetchQueries({ inactive: true });
// inactive한 query를 refetch합니다.
```

### 장점

1. 여러 컴포넌트에서 중복되는 에러 처리를 한번에 처리할 수 있다. 
2. API 통신 에러도 react query같은 라이브러리를 사용하면 한번에 에러 처리를 할 수 있다. 
3. 사용자가 에러때문에 흰 화면을 볼 필요가 없다. 

## 참고

- [suspense 옵션을 사용할 때 주의할 점](https://sangminnn.tistory.com/76)
- [[React] Suspense](https://ko.reactjs.org/docs/react-api.html#reactsuspense)
- [[React] ErrorBoundary](https://ko.reactjs.org/docs/error-boundaries.html#introducing-error-boundaries)
- [[React] error-boundary-how-about-event-handlers](https://ko.reactjs.org/docs/error-boundaries.html#how-about-event-handlers)
- [React-Query-Error-Boundary-적용하기](https://velog.io/@suyeon9456/React-Query-Error-Boundary-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0)
