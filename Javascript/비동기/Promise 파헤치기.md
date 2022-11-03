## Promise 파헤치기

: 비동기 처리 상태와 처리 결과를 관리하는 객체

**콜백 패턴**은

- 콜백 헬로 인해 가독성이 나쁘다.
- 비동기 처리 중 발생한 에러의 처리가 곤란하다.
- 여러개의 비동기 처리를 한번에 처리하는 데도 한계가 있다.

**비동기 처리 중 발생한 에러의 처리가 곤란하다.**

1. 비동기 함수는 코드가 완료되지 않았다고 해도 기다리지 않고 종료된다.
2. 즉, 비동기 동작 코드는 비동기 함수가 종료된 이후에 완료된다.
3. 즉, 비동기 동작 코드의 처리 결과를 비동기 함수에서 외부로 반환하거나, 상위 스코프의 변수에 할당하면 기대한 대로 동작하지 않는다.

예를 들어 setTimeout 함수의 콜백 함수에서 처리 결과로 외부로 반환하거나, 상위 스코프 변수에 할당하면 기대한대로 동작하지 않는다.

```jsx
let global = 0;

setTimeout(() => { global = 100 }, 0});
console.log(global); // 0
```

XMLHttpReqeust를 사용하여 비동기 처리를 하는 코드 안에서

onload 이벤트 핸들러는 비동기적으로 동작한다.

```jsx
// 1️⃣ 반환
const get = url => {
  const xhr = new XMLHttpRequest();

  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      return JSON.parse(xhr.response); // 서버의 응답값 반환
    }
  };
};

const response = get(url);
console.log(response); // undefined

// 2️⃣상위 스코프 변수에 할당

let global;

const get = url => {
  const xhr = new XMLHttpRequest();

  xhr.open('GET', url);
  xhr.send();

  xhr.onload = () => {
    if (xhr.status === 200) {
      global = JSON.parse(xhr.response); // 서버의 응답을 상위 스코프 변수에 저장
    }
  };
};

get(url);
console.log(response); // undefined
```

서버의 응답을 반환한다면?

1. onload 이벤트 핸들러의 반환값은 get 함수의 반환값이 아니다.
2. onload 이벤트 핸들러는 get 함수가 호출하지 않기 때문에 이벤트 핸들러의 반환값을 get이 캐치할 수 없다.

서버의 응답을 상위 스코프 변수에 저장한다면?

1. 이벤트 핸들러는 언제나 console.log가 종료된 이후에 호출되므로 global에는 아직 서버 응답값이 저장되어 있지 않다.
2. 즉, 상위 스코프 변수에 할당하면 처리 순서가 보장되지 않는다.

상위 스코프 변수에 저장되지 않는 이유가 뭘까?

1. get이 호출되면 get의 실행 컨텍스트가 콜 스택에 푸쉬된다.
2. 함수의 코드 실행과정에서 이벤트 핸들러가 바인딩 된다. (onload)
3. get 함수가 종료되면 get의 실행 컨텍스트가 콜 스택에서 팝되고, 곧바로 console.log가 호출된다. (console.log의 실행 컨텍스트가 스택에 푸쉬된다.)
4. 이벤트 핸들러는 즉시 실행되지 않는다. onload 이벤트 핸들러는 load 이벤트가 발생하면 일단 태스크 큐에 저장되어 대기하다가, 콜 스택이 비면 이벤트 루프에 의해 콜 스택으로 푸쉬되어 실행된다.
5. 따라서 이벤트 핸들러가 실행되는 시점에는 콜 스택에 비어있는 상태여야 하므로 console.log가 이미 종료된 이후이다.

즉 비동기 함수에서 비동기 처리 결과를 외부로 반환할 수 없고, 상위 스코프 변수에 저장할 수 없기 때문에

**비동기 처리 결과에 대한 후속 처리는 비동기 함수 내에서 수행해야 한다.**

비동기 함수를 범용적으로 사용하기 위해 처리하는 함수를 콜백함수를 넘기는 것이 일반적이다.

**후속 처리를 콜백 함수에게 맡길 경우**

1. 비동기 처리 결과를 가지고 또 다시 비동기 함수를 호출해야 한다면 콜백 함수 호출의 중첩도가 높아지게 된다. 이를 콜백헬이라고 한다.

```jsx
get(`url/posts/1`, ({ userId }) => { // userId 취득
	get(`url/users/${userId}`, ({ userInfo }) => { // userInfo 취득
			...
	})
})
```

2. 에러 처리가 곤란하다.

```jsx
try {
  setTimeout(() => {
    throw new Error('Error!');
  }, 2000);
} catch (e) {
  console.error(e);
}
```

setTimeout의 콜백 함수가 실행될 때, setTimeout 함수는 이미 콜스택에서 제거된 상태다.

만약 콜백 함수의 호출자(caller)가 setTimeout이라면, 콜 스택의 콜백 함수 실행 컨텍스트의 하위 EC가 setTimeout이여야 하다.

**에러는 호출자(caller) 방향으로 전파된다.** 즉, 콜 스택의 아래 방향으로 전파된다.

따라서 콜백 함수가 발생시킨 에러는 catch 블록에서 캐치되지 않는다.

## Promise의 도입

> ES6에서 도입

- Promise 생성자 함수는 비동기 처리를 수행할 콜백 함수를 인수로 받는데, 콜백함수는 resolve와, reject 함수를 인수로 전달받는다.

```jsx
new Promise((resolve, reject) => {
	if (비동기 성공) {
		resolve('result')
	} else if (비동기 실패) {
		reject(new Error())
	}
})
```

- 프로미스를 사용하면 비동기 처리 결과를 반환해줄 수 있다.

```jsx
const promiseGet.= url => {
	return new Promise((resolve, reject) => {
		const xhr = new XMLHttpRequest();

		xhr.open('GET', url);
		xhr.send();

		xhr.onload = () => {
			if (xhr.status === 200) {
				resolve(xhr.response)
			}
	}
	});
}
// Promise를 반환한다.
```

### 프로미스는 후속 처리를 위한 후속 메서드를 제공한다.

then, catch, finally

- 세 메서드 모두 promise를 반환한다.

**then : 두개의 콜백함수를 인수로 전달 받는다.**

- 첫번째 : fulfilled 상태가 되면 호출된다. 비동기 처리 결과를 인수로 받는다.
- 두번째 : rejected 상태가 되면 호출된다. 프로미스의 에러를 인수로 받는다.

언제나 프로미스를 반환한다.

then 메서드에서 프로미스를 명시적으로 반환하면 그것을 그대로 반환하고,

프로미스가 아닌 값을 반환하면 그 값을 암묵적으로 resolve, reject하는 프로미스를 생성해 반환한다.

**catch : 한개의 콜백함수를 인수로 전달 받는다. 프로미스가 rejected 상태일 경우만 호출된다.**

- then(undefined, onRejected)와 동일하게 동작한다.

**finally : 한개의 콜백함수로 인수를 전달 받는다.**

- promise 상태와 상관없이 공통적으로 수행해야 할 처리 내용이 있을 때 유용하다.

### 프로미스에서의 에러 처리

- then 메서드의 두번째 인자 콜백함수를 이용할 경우 첫번째 콜백함수에서 발생한 에러를 캐치하지 못하고 코드가 복잡해져서 가독성이 좋지 않다.

```jsx
promise(url).then(
  value => {}, // 1
  error => {} // 1번에서 발생한 오류를 캐치하지 못한다.
);
```

- catch 메서드를 모든 then 메서드를 호출한 이후에 호출하면 비동기 처리에서 발생한 에러와, then 메서드 내부에서 발생한 에러까지 모두 캐치할 수 있다. (권장하는 방식)

### 프로미스의 콜백헬 해결

- **프로미스 체이닝** : then, catch, finally 후속 처리 메서드는 언제나 promise를 반환하므로 연속적으로 호출할 수 있다. 이를 프로미스 체이닝이라고 한다.

### 프로미스의 정적 메서드

프로미스는 생성자 함수로 사용되지만, 함수도 객체이므로 메서드를 가질 수 있다.

- Promise.resolve : 이미 존재하는 값을 프로미스로 래핑하여 생성 (즉, resolve된 promise를 생성한다.)
- Promise.reject
- Promise.all : 여러개의 비동기 처리를 모두 병렬 처리할 때 사용한다.
  - 만약 인수 배열의 하나의 promise라도 rejected 상태가 되면 다른 프로미스다 fulfilled 되는 것을 기다리지 않고 즉시 종료한다.
- Promise.race : 모든 promise가 fulfilled 될 때까지 기다리는 것이 아니라 가장 먼저 fulfilled 상태가 된 프로미스의 처리 결과를 resolve하는 프로미스를 반환한다.
- Promise.allSettled : 모든 promise가 settled(fulfilled, rejected) 상태가 되면 처리 결과를 반환한다.

### 마이크로태스크 큐

- promise의 후속 처리 메서드의 콜백함수는 마이크로태스크 큐에 저장된다.
- 마이크로태스크 큐는 태스크 큐보다 우선순위가 높다. (즉, 이벤트 루프가 콜 스택이 비면 우선적으로 마이크로태스크 큐에 대기하고 있는 함수를 가져와 실행한다. 이게 다 비면 태스크 큐)
- 마이크로 태스트 큐에 저장되는 친구들 : promise의 후속 처리 메서드
- 태스크 큐에 저장되는 친구들 : 비동기 함수의 콜백 함수, 이벤트 핸들러

### fetch vs XMLHttpRequest

**XMLHttpRequest 객체**

- 프로미스를 반환하지 않고 이벤트 기반으로 비동기를 처리한다.
- 만약 이벤트 핸들러에서 비동기 결과에 대한 후속 처리를 해주고 싶다면, 이벤트 핸들러에서 콜백함수를 실행 시키거나 XMLHttpRequest 로직를 Promise로 감싸서 반환해줘야 한다.

**이보다 쓰기 편한게 fetch web API**

- 프로미스를 지원한다.
- 즉, 콜백 패턴의 단점에서 ㄷ자유롭다.
- fetch 함수는 Response 객체를 래핑한 Promise 객체를 반환한다.
  - Response 객체는 HTTP 응답을 나타내는 다양한 프로퍼티를 제공한다.
  - 만약 응답의 MIME 타입이 application/json이라면 응답 몸체를 취득하기 위해 Response.prototype.json 메서드를 사용해야 한다. (json 메서드는 Response 객체에서 HTTP 응답 몸체를 취득하여 역직렬화한다.)

> MIME 타입 : 문서의 타입을 알려주는 매커니즘  
> 직렬화 : 객체 → 문자열, 역직렬화 : 문자열 → 객체

**근데 fetch도 문제가 있다.**

- fetch 함수가 반환하는 promise는 기본적으로 404나 500같은 HTTP 에러가 발생해도 에러를 reject하지 않고 response의 ok 프로퍼티를 false로 설정한 Response 객체를 resolve한다. (오프라인 등 네트워크 장애나 CORS 에러에 의해 요청이 완료되지 못한 경우에만 reject한다.)
- 그렇기 때문에 fetch 함수를 사용할 때는 ok 프로퍼티를 명시적으로 확인해야 한다.

```jsx
fetch(wrongUrl)
	.then(() => console.log('ok'); // ok
	.catch(() => console.log('error');
```

```jsx
fetch(wrongUrl).then(() => {
  if (!response.ok) throw new Error();

  return response.json();
});
```

**라이브러리 axios**

- axios는 모든 HTTP 에러를 reject하는 promise를 반환한다. 따라서 모든 에러를 catch에서 처리할 수 있어서 편리하다.
  - 또, interceptor, 요청 설정 등 fetch보다 다양한 기능을 지원한다.
