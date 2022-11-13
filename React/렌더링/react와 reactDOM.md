# react와 reactDOM

react는 컴포넌트와 props, state를 관리하며, 이들의 변경사항을 파악하고, 

reactDOM은 변경사항의 스냅샷을 전달받는다. 

즉, react는 view를 만들어내기 위한 라이브러리이고, reactDOM은 만들어낸 것을 브라우저에 보여주기 위해 사용한다. 

## reactDOM

- 변경 사항 스냅샷을 기반으로 실제 DOM을 변경한다.
- react의 아이디어를 웹 브라우저와 바인딩하는 도구이다.
- react는 브라우저와 아무런 관련이 없다.

## 기타

- react는 브라우저와 전혀 관련이 없기 때문에 react-native 등이 관련되고 있는 이유이다.

## 지원하는 메서드

- createPortal()
- flushSync()
- createRoot() (react-dom/client)

### createPortal

```tsx
ReactDOM.createPortal(그리고 싶은 컴포넌트, real DOM 엘리먼트);
```

- 부모 컴포넌트 DOM 바깥에 있는 DOM 노드로 자식을 렌더링하고 싶을 때 사용
- 특정 DOM노드에 렌더링 하고 싶다면 추천
- portal로 선언된 element라도 부모 컴포넌트로 감싸져 있다면 이벤트 버블링을 부모 컴포넌트에서 포착할 수 있다.
- 전형적인 usecase : 다이얼로그, 호버카드, 툴팁
- 장점 : 부모 컴포넌트와 별개로 그려지기 때문에 부모 스타일에 제약받지 않아도 된다.

### flushSync()

리액트는 setState를 비동기적으로 수행한다. 많은 상태를 업데이트 해도 단 한번만 리렌더링한다. 

setState 사이에서 바로 DOM에 적용하고 싶다면 flushSync를 사용하면 된다. 

간단하게! 보려면 async/await로 생각하면 된다. setState를 동기적으로 작동하게 만든다. 

```tsx
const handleTodo = () => {
	flushSync(() => {
		setTodos([...todos, { newTodo }]);
	})
	listRef.current.scrollTop = listRef.current.scrollHeight;
} 

// 새로운 todo가 들어오면 스크롤을 내리는 코드 
// 하지만 setState가 동기적으로 실행되지 않기 때문에 
// 새로운 todo가 들어오기 전에 스크롤이 내려가 버린다. 
// 이를 flushSync로 해결가능
```

> react-dom/client 패키지는 클라이언트에서 앱을 초기화하는 데 사용되는 클라이언트 별 메서드를 제공한다. 대부분의 컴포넌트는 이 모듈을 사용할 필요가 없다.
> 

### createRoot()

- container를 받고 root를 반환한다.
- root는 react element를 DOM안에 그리기 위해 사용된다. (render 메서드와 함께)

## 참고

- [https://ko.reactjs.org/docs/react-dom.html](https://ko.reactjs.org/docs/react-dom.html)
- [react vs reactDOM](https://velog.io/@yeum0523/React-vs-React-DOM)
- [flushSync](https://kyounghwan01.github.io/blog/React/React18/flushsync/#react18%E1%84%8B%E1%85%B4-%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB-%E1%84%80%E1%85%B5%E1%84%82%E1%85%B3%E1%86%BC-flushsync)
