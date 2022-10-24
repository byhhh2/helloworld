# 클래스 컴포넌트 vs 함수 컴포넌트

## 클래스 컴포넌트의 단점은 이렇다.

1. 로직을 재사용하기 어렵다.
2. [생명주기 메서드는 이해하기 어렵다.](#생명주기-메서드는-이해하기-어렵다)
3. [class 개념이 어렵다.](#class-개념이-어렵다)
4. [class는 render될 때의 값을 유지하지 않을 수 있다.](#class는-render될-때의-값을-유지하지-않을-수-있다)

<br>

---

<br>

## 생명주기 메서드는 이해하기 어렵다.

### 코드 중복을 야기한다.

클래스 컴포넌트는 관련있는 로직을 여러개의 메서드에 나누어 놓는 경우가 자주 있다.

리액트의 클래스 컴포넌트는 마운트 단계와 업데이트 단계를 통 틀어서 수행할 수 있는 메서드가 존재하지 않는다.

만약 마운트 단계와 업데이트 단계에서 같은 코드를 수행하길 원한다면 중복해서 작성해주는 방법밖에 존재하지 않는다.

```jsx
componentDidMount() {
  document.title = `You clicked ${this.state.count} times`;
}

componentDidUpdate() {
  document.title = `You clicked ${this.state.count} times`;
}

// 둘은 같은 코드임에도 불구하고 생명주기가 다르다는 이유로
// 중복되어 작성되어야 한다.
```

```jsx
// 만약에 함수 컴포넌트의 hook으로 작성되었다면?

useEffect(() => {
  document.title = `You clicked ${this.state.count} times`;
});
```

<br>
<br>

### 관심사를 구분하기 어렵다.

클래스 컴포넌트의 생명주기 메서드들은 관련 없는 로직들을 모아 놓는 경우가 많다.

`componentDidMount`에서 이벤트 리스너를 설정하는 것과 같은 관계없는 로직이 포함되기도 하고 `componentWillUnmount`에서 cleanup 로직을 수행하기도 된다.

즉, 연관없는 코드들이 단일 메서드로 결합된다.
이는 버그가 쉽게 발생하고 무결성을 쉽게 해친다.

```jsx
componentDidMount() {
	// (1)
  document.title = `You clicked ${this.state.count} times`;
  // (2)
	ChatAPI.subscribeToFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}

// (1) document의 title을 변경하는 로직과
// (2) 상태를 구독하는 로직은
// 전혀 관계 없는 로직임에도 불구하고 같은 메서드에 존재한다.

componentDidUpdate() {
  document.title = `You clicked ${this.state.count} times`;
}

componentWillUnmount() {
  ChatAPI.unsubscribeFromFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  ); // (3)

// 상태를 구독 해제하는 로직은
// 생명주기가 다르다는 이유로 (2) 상태를 구독하는 로직과
// 따로 떨어져야 한다.
}

```

함수 컴포넌트같은 경우 hook을 통해 비슷한 것을 하는 작은 묶음으로 로직을 분리할 수 있다.

```jsx
useEffect(() => {
  document.title = `You clicked ${this.state.count} times`;
});
// document의 title을 변경하는 로직 useEffect

useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
});
// 구독을 진행하고 해제하는 로직 useEffect
```

hook을 사용하면 **생명주기 메서드에 따라서가 아니라 코드가 무엇을 하는지에 따라 나눌 수 있다.**

<br>
<br>

## class 개념이 어렵다.

- class에서는 `this` 키워드를 활용해야 한다.
- js의 `this` 키워드는 다른 언어와 다르게 작동하여 사용자에게 큰 혼란을 준다. 이는 코드 재사용성과 구성을 매우 어렵게 만든다.
- js의 `this`는 함수가 호출되는 방식에 따라 동적으로 결정된다. ⇒ 이런 이유로 이벤트 핸들러가 등록되는 방법을 정확히 파악해야 한다.

```jsx
class Basic extends Component {
  constructor(props) {
    super(props);

    this.state = {
      bool: false,
    };

    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => {
      bool: !state.bool;
    });

    // bind를 하지 않으면 여기서 this가
    // 전역 객체를 가르킨다.
  }

  render() {
    return <button onClick={this.handleClick}>toggle</button>;
  }
}
```

```jsx
// 만약 화살표 함수를 사용한다면 bind 없이 사용할 수 있다.
// 화살표 함수 내부의 this는 상위 스코프의 this를 가르킨다.

class Basic extends Component {
  constructor(props) {
    super(props);

    this.state = {
      bool: false,
    };

    // this.handleClick = this.handleClick.bind(this);
  }

  handleClick = () => {
    this.setState(state => {
      bool: !state.bool;
    });
  };

  render() {
    return <button onClick={this.handleClick}>toggle</button>;
  }
}
```

<br>
<br>

## class는 render될 때의 값을 유지하지 않을 수 있다.

props는 리액트에서 불변(immutable)한 값이다.
하지만 this는 변경 가능(mutable)하고, 조작할 수 있다.
(이것이 this가 클래스에 존재하는 목적이다.)

요청이 진행되고 있는 상황에서 컴포넌트가 다시 rendering된다면
this.props 또한 바뀐다.

여기 두가지 버전의 컴포넌트가 있다.  
로직은 간단하다.

> Follow 버튼을 누르면 props를 통해 전달된 user를 Followed 했다는 메시지를 3초 뒤에 alert한다.

만약 user가 Dan이라면 버튼을 누르고 3초뒤에 Followed Dan이라는 창이 띄워진다.

```jsx
// 1️⃣ 함수 컴포넌트로 작성

function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return <button onClick={handleClick}>Follow</button>;
}

// 2️⃣ 클래스 컴포넌트로 작성

class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };

  handleClick = () => {
    setTimeout(this.showMessage, 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

둘의 같은 로직을 가지고 있지만  
차이가 있다.

[Live demo](https://codesandbox.io/s/pjqnl16lm7)를 실행해보면 이 차이를 알아챌 수 있다.

![image](https://user-images.githubusercontent.com/52737532/197569560-d084b5d1-849d-49fb-bbad-952e1f347cf2.gif)

1. Dan에서 Follow 버튼을 누른다.
2. 3초가 지나기 전에 선택된 프로필을 바꾼다.
3. alert을 확인한다.

클래스 컴포넌트와 함수 컴포넌트의 실행 차이는

**함수 컴포넌트** : Dan을 follow 한 후 3초가 지나기 전에 sophie의 프로필로 이동하면 알림창에 `Followed Dan` 이 쓰여있다.

**클래스 컴포넌트** : Dan을 follow 한 후 3초가 지나기 전에 sophie의 프로필로 이동하면 알림창에 `Followed Sophie` 가 쓰여있다.

사실 `Dan → Sophie`로 프로필을 이동했을 때 Dan이 팔로우되는 것이 옳은 로직이다.  
사용자가 어떤 사람을 팔로우하고, 다른 사람의 프로필로 이동했다 하더라도 컴포넌트가 이를 헷갈려서는 안되기 때문이다.  
클래스 컴포넌트로 구현한 로직은 명백한 버그다!

<br>

#### 클래스는 왜 이렇게 동작할까?

`this`는 변경 가능하다. (mutable)  
this가 변경 가능하기 때문에 life cycle 메서드를 호출할 때 업데이트된 값을 읽어올 수 있다.

요청이 진행되고 있는 상황에 컴포넌트가 re-render된다면 this.props 또한 변경된다.  
props와 state는 render에 종속된다.  
하지만, this.props를 setTimeout에게 넘겨준 callback 함수가 호출하게 되면서 이 종속관계가 깨지게 된다.

props와 state는 더이상 이전 render에 종속되지 않고 올바른 props를 잃어 버린다. (즉, 바뀌어 버린 props를 새로 가져온다.)

이와 다르게 함수 컴포넌트에서는 값이 인자로 전달됐기 때문에 props는 보존된다. this와 다르게, 함수가 받는 인자는 리액트가 변경할 수 없다.

함수 컴포넌트에서 다른 props를 통해 다시 함수 컴포넌트를 render하면 리액트는 함수 컴포넌트를 **새로 호출한다.** 이런 이유로 props는 이전 render에 안전하게 종속되어 있을 수 있다.

<br>
<br>

## 참고

- [함수형 컴포넌트와 클래스, 어떤 차이가 존재할까?](https://overreacted.io/ko/how-are-function-components-different-from-classes/)
- [Using the Effect Hook](https://ko.reactjs.org/docs/hooks-effect.html)
- [Hook의 개요](https://ko.reactjs.org/docs/hooks-intro.html#complex-components-become-hard-to-understand)
