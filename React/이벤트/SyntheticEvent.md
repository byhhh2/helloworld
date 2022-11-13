## SyntheticEvent

React는 이벤트 시스템 일부를 구성하는 SyntheticEvent Wrapper를 사용한다.

- 이벤트 핸들러는 **모든 브라우저에서 이벤트를 동일하게 처리하기 위한** SyntheticEvent 객체를 전달 받는다.
    - 브라우저 고유 이벤트와 같지만, 모든 브라우저에서 동일하게 동작한다.
    - W3C 명세에 따라 합성 이벤트를 정의하기 때문에 브라우저 호환성에 대해 걱정할 필요가 없다.
- 브라우저 고유 이벤트가 필요하다면 nativeEvent 속성을 참조하면 된다.
    - SyntheticEvent는 브라우저 고유 이벤트가 직접 대응되지 않는다.
    - `onMouseLeave` 에서 `e.nativeEvent`는 `mouseout` 이벤트를 가르킨다.
- 캡처링 단계에서 이벤트 핸들러를 등록하기 위해서는 이벤트 이름에 Capture를 덧붙이면 된다.
    - `onClick` → `onClickCapture`
- DOM 엘리먼트가 생성된 후 리스너를 추가하기 위해 `addEventListener`를 호출할 필요가 없다. 엘리먼트가 처음 렌더링될 때 리스너를 제공하면 된다.

> **Wrapping**. 
> 
> 기본 기능을 감싸 새로운 기능을 만드는 것   
> 

## 참고

- [https://ko.reactjs.org/docs/events.html](https://ko.reactjs.org/docs/events.html)
- [https://ko.reactjs.org/docs/handling-events.html](https://ko.reactjs.org/docs/handling-events.html)
- [https://abangpa1ace.tistory.com/129](https://abangpa1ace.tistory.com/129)
