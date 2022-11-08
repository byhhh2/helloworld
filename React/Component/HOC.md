## **고차 컴포넌트**

- 컴포넌트 로직을 재사용하기위해 컴포넌트를 인자로 받고, 새 컴포넌트를 반환한다.
- 네이밍할 때 with을 prefix로 붙인다.

> **횡단 관심사** (= 공통 관심사) 
>   
> 어플리케이션 각 계층에서 공통적으로 필요한 문제 
>  
> 
> ```jsx
> <App>
> 	<Router>
> 		<Page>
> 
> // Router 컴포넌트에서도 주소 정보가 필요하고, Page에서도 주소 정보가 필요하다면?
> // 두 컴포넌트 둘 다 주소 이동 기능을 가질 수 있다. 
> // 즉, 각 계층을 넘어 공통으로 필요한 관심사 
> ```
> 
> 예시 : 주소 정보, 전역 스토어, 로깅 등 
> 

## 클래스 컴포넌트에서 HOC가 없을 때의 문제

1. 코드가 중복된다. 
2. 로직이 컴포넌트 고유 역할이 아닐 확률이 높다. (어디에도 소속되어 있지 않다. 전반에 사용되는 공통 기능이기 때문)

```jsx
// Header 컴포넌트

class Header extends React.Component {
	componentDidMount() {
		console.log('Header is mounted')
	}
}

// Button 컴포넌트 

class Button extends React.Component {
	componentDidMount() {
		console.log('Button is mounted')
	}
}

// 코드가 중복되고, 컴포넌트 고유 역할이 아니다. 

```

## HOC로 로직 재활용하기

로깅함수를 WrapperComponent에 선언해주면 인자로 들어오는 component들이 재활용 가능해진다. 

간단하게 구조를 생각해보면

1. withLogging은 새로운 컴포넌트(WrapperComponent)를 반환한다. 
2. WrapperComponent안에는 재사용가능한 로직이 있고, 인자로 들어오는 Component를 render한다. 
3. Component는 재사용한 로직을 props로 받아서 사용할 수 있게 된다. 

```jsx
const withLogging = <T,>(Component: React.ComponentType<T>) => {
  function WrapperComponent(props: T) {
    const logging = () => {
      //
    };
    const wrappeedProps = {
      logging,
    };

    return <Component {...props} {...wrappeedProps} />;
  }

  return WrapperComponent;
};

export default withLogging;

// withLogging : 컴포넌트를 반환한다.
// WrapperComponent : 인자로 받은 컴포넌트를 render한다. 

// 다시 사용하고 싶은 로직을 WrapperComponent안에 작성하면 된다. 
```

### 컴포넌트마다 특정 값이 필요하다면?

한번 더 함수로 감싸서 전달해주거나, withLogging에서 인자를 두개 받아주면 된다. 

```jsx
// 함수로 한번 더 감싸기 

const withLogging =
  (componentName: string) =>
  <T,>(Component: React.ComponentType<T>) => {
    function WrapperComponent(props: T) {
      const logging = () => {
        console.log(componentName + ' is mounted');
      };
      const wrappeedProps = {
        logging,
      };

      return <Component {...props} {...wrappeedProps} />;
    }

    return WrapperComponent;
  };

export default withLogging;
```

## 활용 방안

- class component에서 로직 재사용
- class component에서 hook을 쓰고 싶다면

## 고차 컴포넌트를 만들 때 주의할 점

- render 메서드 안에서 고차 컴포넌트를 사용하지 말 것

```jsx
render() {
  // render가 호출될 때마다 새로운 버전의 EnhancedComponent가 생성됩니다.
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 때문에 매번 전체 서브트리가 마운트 해제 후 다시 마운트 됩니다!
  return <EnhancedComponent />;
}
```

react의 비교 알고리즘 재조정은 컴포넌트의 identity를 가지고 트리를 업데이트 할 지, 버릴 지 결정한다. 
render에서 반환된 컴포넌트가 이전 컴포넌트와 동일하다면 이전과, 현재를 비교해서 업데이트만 한다. 
하지만, 동일하지 않으면 버리고 다시 그린다. 즉, 성능상에 문제가 발생한다. 
컴포넌트 정의 바깥에서 HOC를 적용해서 컴포넌트가 한 번만 생성되도록 해야한다. 

- 정적 메서드는 따로 복사해야 한다.

```jsx
// 정적 함수를 정의합니다
WrappedComponent.staticMethod = function() {/*...*/}
// HOC를 적용합니다
const EnhancedComponent = enhance(WrappedComponent);

// 향상된 컴포넌트에는 정적 메서드가 없습니다.
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

HOC를 적용하면, 기존 컴포넌트는 새로운 컴포넌트로 감싸진다.
즉, 새로운 컴포넌트는 정적 메서드를 가지고 있지 않다. 

```jsx
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // 복사 할 메서드를 정확히 알아야 합니다.
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
} 
```

그러므로 새로운 컴포넌트를 반환하기 전에 정적 메서드를 복사 해주어야 한다.

- ref는 전달되지 않는다.

리액트에서는 ref가 실제 props이 아니라 key처럼 특별 취급된다. 
ref를 전달하고 싶다면 forwardRef를 사용해야한다. 

## 참고

- [[react] HOC](https://ko.reactjs.org/docs/higher-order-components.html)
- [리액트 고차 컴포넌트란?](https://itprogramming119.tistory.com/entry/React-%EA%B3%A0%EC%B0%A8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EB%9E%80)
- [tsx에서 generic쓰기](https://www.ashbyhq.com/blog/engineering/generic-arrow-function-in-tsx)
