# useState는 어떻게 값을 유지할까? 

- 함수 컴포넌트는 re-render시 다시 실행된다.
- 일반적으로 일반 변수는 함수가 끝날 때 사라지지만, state 변수는 React에 의해 사라지지 않는다.
- `useState`는 JS의 클로저 개념을 통해 변수 값을 유지시킨다.

> **Closure**
> 
> 
> 외부 함수보다 중첩 함수가 더 오래 유지되는 경우, 중첩 함수는 외부 함수의 변수를 참조할 수 있다. 
> 
> **Garbage Collection**
> 
> 최신 브라우저는 ‘표시하고-쓸기 알고리즘’을 사용하는데, 
> 
> 더 이상 필요하지 않은 메모리를 해제할 때  
> 
> 더 이상 닿을 수 없는 오브젝트를 기준으로 한다. 
> 

## `useState`

### useState는 어떻게 state 값을 유지할까?

```jsx
// useState를 사용하는 목적은 함수 컴포넌트가 재실행되어도 state값을 유지하기 위해서다. 
// 1. 값이 어떻게 유지가 되는가?
// _val을 계속적으로 참조하고 있다. 

const React = (function () {
  let _val;
      
  function useState(initialVal) { // useState
    const state = _val || initialVal;
    const setState = (newVal) => {
      _val = newVal;
    };

    return [state, setState];
  }
  
  function render(Component) { // render 
    const C = Component(); // re-render시 마다 Component() 함수를 실행한다. 
    C.render();
    return C;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  // count -> state -> _val 순으로 참조하고 있기 때문에 _val은 garbage collection되지 않는다. 
  
  return {
    render: () => console.log(count),
    click: () => setCount(count + 1),
  };
}

// App에는 { render, click }이 존재하고 계속 count는 참조되고 있기 때문에 _val은 garbage collection되지 않는다. 
var App = React.render(Component);
App.click();
var App = React.render(Component); 
App.click();
var App = React.render(Component);
```

- state의 원본 변수인 _val가 지속적으로 사용되며 garbage collection되지 않기 때문에 state가 유지될 수 있다.

### 여러 개의 useState를 어떻게 사용할까?

```jsx
const React = (function () {
  let hooks = [];
  let idx = 0;
      
  function useState(initialVal) { // useState
    const state = hooks[idx] || initialVal;
    const _idx = idx; // setState 안의 idx가 useState에 의해서 변하지 않게 freeze한다.
		// 임시변수를 사용해 각 setState가 자신만의 _idx를 갖게 한다. 

    const setState = (newVal) => {
      hooks[_idx] = newVal; 
    };

    idx++;
    return [state, setState];
  }
  
  function render(Component) { // render 
    idx = 0; // render 될 때마다 idx가 증가하는 경우 방지 
    const C = Component(); 
    C.render();
    return C;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState("apple");
  
  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  };
}

var App = React.render(Component); 
// React.render의 실행으로 idx는 0
// Component의 실행 
// count의 useState 실행으로 _idx는 0, idx는 1
// text의 useState 실행으로 _idx는 1, idx는 2 
App.click(); // setCount의 실행 
// setCount(count + 1), setCount는 _idx가 0이므로 hooks[0]의 값이 count + 1로 변경된다. 
// hooks : [ 2 ]
var App = React.render(Component); 
// React.render의 실행으로 idx는 0
// Component의 실행 _
// count의 useState 실행으로 _idx는 0, idx는 1
// text의 useState 실행으로 _idx는 1, idx는 2 
App.type("orange");
// setText('orange'), setText는 _idx가 1이므로 hooks[1]의 값이 'orange'로 변경된다. 
// hooks : [ 2, 'orange' ]
var App = React.render(Component);
```

- 값들을 배열에 저장하고 index를 통해 조정한다.

> **React에서 Hook을 조건문 내에 두지 말라는 이유는?**
> 
> 
> ```jsx
> function Component() {
>   if (Math.random() > 0.5) {
>     const [count, setCount] = React.useState(1); // ❌
>   }
> 
>   const [text, setText] = React.useState("apple");
> 
>   return {
>     //...
>   };
> }
> ```
> 
> `count`의 index는 `0`, `text`의 index는 `1`이어야 한다. 
> 
> 하지만 조건적으로 `text`의 index를 0 또는 1로 할 경우 
> 
> 매 렌더링 때마다 순서가 보장되지 않는다. 
> 

## 참고

- [https://ingg.dev/hook-work/](https://ingg.dev/hook-work/)
