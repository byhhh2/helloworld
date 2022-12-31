# React 18 둘러보기

1. useTransition 
2. Suspense를 지원하는 SSR
3. Automatic Batching 

## useTransition, startTransition

### 어떤 문제를 해결하고자 하는가?

> **블로킹 렌더링 문제**
> 
> 
> 한번 렌더링 연산이 시작되면 멈출 수 없다. 
> 
> 대형 화면 업데이트의 경우 렌더링이 되는 동안 페이지 지연 발생 
> 

**문제 예시** 

(onChange 이벤트처럼 타이핑을 할 때마다 re-render되는 경우를 생각해보자.) 

input 창에 입력되는 텍스트의 길이에 비례해서 파란색 박스가 계속 출력된다고 생각해보자. 

<img width="336" alt="스크린샷 2022-12-30 오후 10 46 14" src="https://user-images.githubusercontent.com/52737532/210081782-386650bb-d7c6-4c57-ad44-2216cd10bb0b.png">

만약 입력이 매우 많게 되면 input창에 인터렉션이 없을 수 있다. 

> **synchronous rendering** 중에는 한번 rendering이 시작되면 결과물이 화면에 보일 때까지 중단될 수 없지만 **concurrent**에서는 rendering이 중단될 수 있다. 이런 이유로 특정 부분의 rendering의 우선 순위 변경시킬 수 있는 **startTransition**이 등장한다.
> 

> **동시성이란?**
> 
> 
> <img width="287" alt="스크린샷 2022-12-30 오후 11 02 52" src="https://user-images.githubusercontent.com/52737532/210081828-41826748-d17f-4d88-8d3d-2d7c5505f92e.png">
> 
> 2개 이상의 작업을 나눠서 동시에 실행되는 것처럼 프로그램을 구조화하는 방법 
> 

### startTransition

- rendering 우선순위를 낮출 수 있다.

타이핑이나 클릭같은 다이렉트 상호작용은 **긴급 업데이트 (urgent updates)**가 되지 않으면 버벅거리며 앱이 이상하다는 느낌을 받을 수 있다. 

이에 반해 화면 결과값은 바로 보여질 것이라고 기대되지 않기 때문에 **전환 업데이트 (transition updates)**는 느리게 업데이트가 되어도 괜찮다. 

```jsx
import { startTransition } from 'react';

setInputValue(input); // 긴급한 업데이트 (urgent updates) 
// 입력같은 경우 지연되면 버그로 판단될 수 있다. 

startTransition(() => {
  setBoxCount(input); // 전환 업데이트 (transition updates) 
	// 화면 결과값은 바로 보여질 것이라고 기대되지 않는다. 
});
```

### useTransition

```jsx
const [isPending, startTransition] = useTransition();
```

- isPending을 통해 transition 작업이 pending 상태인 것을 알 수 있다.

<img width="629" alt="스크린샷 2022-12-30 오후 10 56 49" src="https://user-images.githubusercontent.com/52737532/210081865-bb51a9b9-c3ee-4fa0-b33e-d173e155fc69.png">

이를 조건부 렌더링에 사용하여 pending 상태임을 UI로 나타낼 수 있다. 

> **디바운스와의 차이?**
> 
> - 디바운스는 입력이 종료되어야 화면 업데이트가 가능하다.
> - 하지만 입력이 계속적이면 화면 업데이트가 계속 미뤄지게 된다.
> - 디바운스는 단순히 문제를 뒤로 미루는 것이라고 볼 수 있다.
> 
> <img width="577" alt="스크린샷 2022-12-30 오후 10 58 44 (1)" src="https://user-images.githubusercontent.com/52737532/210081920-550467dd-7e69-4c94-a2ea-8838fd6e0a9a.png">
> 
> **스로틀과의 차이?**
> 
> <img width="577" alt="스크린샷 2022-12-30 오후 10 59 00" src="https://user-images.githubusercontent.com/52737532/210081900-04af9634-329a-4512-ae44-ec279c25b5cf.png">
> 
> - 주기적으로 화면 업데이트를 할 수 있다.
> - 하지만 주기동안 입력을 띄엄띄엄하게 된다면 빈 시간동안 의미없이 기다리는 시간이 생긴다.
> 
> <img width="577" alt="스크린샷 2022-12-30 오후 11 00 02" src="https://user-images.githubusercontent.com/52737532/210127789-6b3fb4e4-e996-401a-97d7-c396b22f5ee7.png">
> 
> **startTransition?**
> 
> <img width="577" alt="스크린샷 2022-12-30 오후 11 00 37" src="https://user-images.githubusercontent.com/52737532/210127876-62b570a5-5ac6-48b8-b899-7add7a292896.png">
> 
> - 화면 업데이트 중에도 사용자 입력을 받을 수 있다.
> - 비어있는 시간이 사라진다. (사용자 경험에 더 좋다.)

### startTransition의 원리?

<img width="389" alt="스크린샷 2022-12-30 오후 11 01 33" src="https://user-images.githubusercontent.com/52737532/210127882-a9516115-4ec4-40cc-a194-dbca328e6ac5.png">

- rendering 도중에도 더 급한 일이 생기면 그것을 먼저 할 수 있다.

## Suspense를 지원하는 SSR

### 어떤 문제를 해결하고자 하는가?

**SSR의 동작방식 ?**

<img width="479" alt="스크린샷 2022-12-30 오후 11 09 05" src="https://user-images.githubusercontent.com/52737532/210127888-4efd4a96-8fa2-417b-bb2e-bc6b49ba2b60.png">

1. 서버에서에 전체 어플리케이션에 대한 데이터를 가져온다. 
2. HTML로 렌더링한다. ⇒ 브라우저에서 보여진다. 
3. 사용자가 페이지 컨텐츠를 볼 수 있다. 
4. 하지만 아직 인터렉티브한 상태가 아니다. 
5. 자바스크립트 코드를 로드한다. 
6. JS 코드를 HTML에 연결한다. (Hydration) 
7. 사용자가 앱과 상호작용할 수 있다. 

Hydration : 컴포넌트를 렌더링하고 이벤트 핸들러를 연결하는 과정  

**기본 SSR의 문제점 ?**

- 모든 data fetch가 끝나야 어떤 것이라도 보여줄 수 있다.
- 모든 자바스크립트 코드를 로딩하기 전에는 Hydration 단계로 넘어갈 수 없다.
- 앱이 상호 작용할 수 있는 상태가 되려면 앱 전체가 Hydration이 완료되어야 한다.

즉, 앱 전체가 각각의 단계를 완료해야 다음의 단계로 넘어갈 수 있다. 

> **Suspense**
> 
> 
> Suspense는 리액트 16에서 도입되었다. 
> 
> 하지만 React.lazy를 통한 코드 스플리팅만 지원했으며 SSR은 지원하지 않았다. 
> 
> 리액트 18에서 SSR에 대한 지원을 추가한다. 
> 

### Suspense + SSR

페이지의 각 컴포넌트들을 Suspense로 묶어서 따로 처리할 수 있다. 

<img width="341" alt="스크린샷 2022-12-30 오후 11 14 41" src="https://user-images.githubusercontent.com/52737532/210127907-e9510250-6d32-42be-a705-fadba2aabd60.png">

<img width="575" alt="스크린샷 2022-12-30 오후 11 14 54" src="https://user-images.githubusercontent.com/52737532/210127909-9b6a2d8d-83f4-42d0-80d1-3029e37d1a7a.png">

### 모든 data fetch가 끝나야 어떤 것이라도 보여줄 수 있다.

<img width="629" alt="스크린샷 2022-12-30 오후 11 15 47" src="https://user-images.githubusercontent.com/52737532/210127912-6e6c8f29-ab23-4c19-a50d-89d8d8985f7a.png">

- 컴포넌트별로 나눠서 data fetch를 진행하고, 특정 컴포넌트에서 data fetch가 되지 않았다면 fallback 화면을 보여준다.
- fallback 화면은 서버에서 rendering에 완료되면 대체된다.

### 모든 자바스크립트 코드를 로딩하기 전에는 Hydration 단계로 넘어갈 수 없다. (선택적 Hydration)

- lazy와 suspense를 SSR에 사용할 수 있다.
- 아직 준비되지 않은 컴포넌트 이외 컴포넌트에서 hydration을 진행할 수 있다.
- Comments 부분의 JS 코드가 모두 로드되면 Comments 부분도 hydration을 시작한다.

<img width="629" alt="스크린샷 2022-12-30 오후 11 23 13" src="https://user-images.githubusercontent.com/52737532/210127918-c13b3e00-7a4d-4612-a1c2-000ba2e245c7.png">

### 앱이 상호 작용할 수 있는 상태가 되려면 앱 전체가 Hydration이 완료되어야 한다.

<img width="336" alt="스크린샷 2022-12-30 오후 11 26 03" src="https://user-images.githubusercontent.com/52737532/210127925-3a120ce3-e735-4874-9e91-fe2aa19a862e.png">

만약 Sidebar, Commnets의 HTML은 모두 로드되었지만, 

JS 코드가 로드되지 않았다면 

React는 Sidebar부터 hydration을 시도한다. 

하지만 만약 사용자가 Comments에서 click같은 상호작용을 시작한다면

React는 Sidebar의 hydration을 멈추고 Comments의 hydration을 진행한다. 

## Automatic Batching

> **Batching이란?**
> 
> 
> 여러 개의 state 업데이트를 하나의 re-render만 발생하도록 그룹화한 것 
> 

### 어떤 문제를 해결하고자 하는가?

이전에는 batching을 react의 이벤트 핸들러에서만 지원했다. 

promise, setTimeout, native event handler에서 setState를 할 경우 배칭이 되지 않았다. 

### 넓어진 batching의 지원 범위

```jsx
// Before: 배칭이 되지 않아서 상태 업데이트가 그룹핑 되지 않고 매번 리렌더링 된다
// (두번 리렌더링 된다)
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
}, 1000);

// After: promise, setTimeout, native 이벤트 핸들러 등에서도 배칭이 된다.
// (한번만 리렌더링 된다)
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
}, 1000);
```

## 참고

- [https://yrnana.dev/post/2022-04-12-react-18](https://yrnana.dev/post/2022-04-12-react-18)
- [[테코톡] 코이의 React 18에서의 변경점](https://www.youtube.com/watch?v=focpJqfSu4k)
