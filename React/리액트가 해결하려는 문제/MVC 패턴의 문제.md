# MVC 패턴의 문제

## MVC 패턴이란?

MVC 패턴은 비즈니스 로직과 화면 (UI 로직)을 구분하는데 중점을 두고 있다.  
즉, `관심사를 분리하여, 역할을 잘 나눠 코드를 관리하자!`에 중심을 두고 있다.

![](https://developer.mozilla.org/en-US/docs/Glossary/MVC/model-view-controller-light-blue.png)

from [MDN](https://developer.mozilla.org/ko/docs/Glossary/MVC)

간단히 설명하자면, 이렇다!

|     관심사     |            하는 일            |                                                                  이동                                                                  |
| :------------: | :---------------------------: | :------------------------------------------------------------------------------------------------------------------------------------: |
|   **Model**    | 데이터와 비즈니스 로직을 관리 |                                       - Model -> view : 데이터 상태가 변경되면 View에게 알린다.                                        |
|    **View**    |     레이아웃과 화면 처리      |                                         - Model -> View : 표시할 데이터를 Model로부터 받는다.                                          |
| **Controller** |                               | - View -> Controller : 사용자에게 입력을 받는다. <br> - Controller -> View / Model : 입력에 대한 응답으로 View / Model을 업데이트한다. |

MVC 패턴의 장점은 아래와 같다.

- 비즈니스 로직과 UI 로직이 분리되어 유지보수를 독립적으로 수행 가능하다. (View, Model, Controller가 독립되어 있어 유지보수를 독립적으로 수행 가능)

<br>
<br>

MVC 패턴의 장점은 패턴 자체가 **이상적으로 구현**되었을 때의 얘기이다.
사실상 Model과 View의 완벽한 분리자체가 실제 구현에서는 어렵다.

이유는, 데이터와 UI를 연결하는 코드는 필수적이기 때문이다!

<br>

[MDN](https://developer.mozilla.org/ko/docs/Glossary/MVC)을 참고하면 Model이 View를 업데이트하는 화살표는 있어도  
View가 Model을 업데이트 하는 화살표는 존재하지 않는다.
하지만 구현에 있어서는 View가 Model을 직접 업데이트 하는 경우가 있다.

페이스북에서 구현한 MVC 패턴만 봐도 View가 Model을 직접적으로 업데이트하는 것을 볼 수 있다.

![](https://user-images.githubusercontent.com/52737532/197575238-8c764eff-c6f4-4157-a40e-ea860306c5f1.png)

이를 **양방향 데이터 바인딩**이라고 한다.  
(즉, Model -> View, View -> Model이 가능하다.)

양방향 데이터 바인딩에는 여러 문제가 존재하는데,  
데이터 흐름이 양방향으로 이뤄지기 때문에 복잡한 프로그램같은 경우에는 프로그램 흐름의 예측가능성이 떨어지게 된다.

![](https://user-images.githubusercontent.com/52737532/197575832-87fb851e-df65-435f-852a-22bf8d2a9fc8.png)

사진에서 보면 하나의 Model을 여러 View가 참조하는 것을 볼 수 있다.  
여러 View에서는 여러 이벤트가 발생하여 Model을 업데이트하려고 할 것이고,  
언제, 어떤 이벤트가 발생하여 Model을 업데이트 했는지 추적하는 것이 쉽지 않게 된다.

<br>

여기서 **단방향 데이터 바인딩**개념이 등장한다.
단방향 데이터 바인딩에서는 View에서 Model로의 직접적인 데이터 갱신이 불가능하다.

> `<input>` 태그로 예시를 들자면,  
> 사용자가 `<input>` 태그에 무엇을 입력할 때 `<input>` 태그의 값이 바뀌는 것이 아니라 `<input>` 태그의 변경을 감지해서 JS 변수 값을 변경시키고 이를 View에 뿌려주는 형식이라고 볼 수 있다.

<br>
<br>

## 참고

- [데이터 바인딩 이해하기](https://adjh54.tistory.com/49)
- [Flux 패턴이란?](https://velog.io/@andy0011/Flux-%ED%8C%A8%ED%84%B4%EC%9D%B4%EB%9E%80)
