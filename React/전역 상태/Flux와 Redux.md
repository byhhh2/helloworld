# Flux와 Redux

## Flux란 무엇인가?

### MVC 패턴의 한계

![Untitled (17)](https://user-images.githubusercontent.com/52737532/210342936-7d7f782f-d198-4914-9a94-6f8a6da78a6f.png)

Flux는 **MVC패턴의 한계**로 인해 등장하게 되었다. 

MVC 패턴은 비즈니스 로직과 화면을 구분하는데 중점을 두고 있다. 즉, 관심사 분리를 위해 나타난 디자인 패턴이다. 

MVC 패턴은 어플리케이션이 복잡해질수록 복잡해지는데 하나의 View에서 발생한 상호작용 때문에 여러 Model이 변경되기도 하고, 하나의 Model이 여러 View에 데이터를 전달하는 상황이 발생한다. Model에 많은 View가 영향을 받을 경우 Model에서 발생한 event가 어플리케이션 전체로 퍼져나가는 것처럼 된다. 이 경우 이벤트로 발생할 **결과를 예측하기 어려워진다.** 

MVC 패턴처럼 View ↔ Model 양방향으로 업데이트가 가능한 구조를 **양방향 데이터 바인딩**이라고 한다. 

> **데이터 바인딩**
> 
> 
> 화면에 보여지는 데이터 (View)와 메모리(JS)에 존재하는 데이터 (Model)을 서로 동기화 시키는 것 
> 

### Flux의 등장

![Untitled (18)](https://user-images.githubusercontent.com/52737532/210342974-b8d0e3b6-2265-4282-bda8-7dd92c47bf19.png)

Flux는 페이스북에서 고안한 패턴으로 MVC 패턴의 한계를 극복하고자 한다. 

MVC 패턴처럼 양방향으로 데이터가 흐르게 되면 이를 예측하기 어렵다는 것을 깨닫고, Flux 패턴은 단방향 데이터 흐름을 고안하게 된다. 

단방향으로 데이터가 흐르게 됨으로써 어플리케이션의 구조와 상태를 심플하게 파악할 수 있게 되었다. 

## Flux의 구성 요소과 흐름

### 구성 요소

- **store**
    - 상태를 저장하는 객체
    - dispatcher를 통해 상태 갱신 명령을 받아 갱신한다.
- **action**
    - 이벤트 타입과 상태를 갱신하기 위한 정보를 담은 객체
    - dispatcher로 보내진다.
- **dispatcher**
    - 어플리케이션 내의 모든 action을 받는다.
    - store에게 action 내용에 따라 상태를 변경하라고 명령한다.
    - 오직 하나만 존재한다.
- **view**
    - store에서 보내주는 상태를 사용자에게 보여준다.
    - 사용자에게 action을 받는다.

### 흐름

![Untitled (19)](https://user-images.githubusercontent.com/52737532/210343030-5d8966e1-873d-4a38-ad14-a2b27ecb291c.png)

1. 사용자가 View를 조작하여 Action이 발생한다. 
2. 갱신할 내용이 담긴 Action이 Dispatcher에 전달된다. 
3. Dispatcher가 Store에게 Action을 전달하며 상태 갱신을 명령한다. 
4. Store가 상태를 갱신한다. 
5. 갱신된 상태를 View에게 전달한다. 

## Redux는 무엇인가?

### Flux와 Redux의 차이는?

Redux는 Flux 아키텍처에 Reducer를 결합한 것이다. 

> Redux = Reducer  + Flux
> 

Flux의 특징은 아래와 같다. 

- Flux는 단 하나의 Dispatcher를 가진다.
- Flux는 여러 개의 Store를 가질 수 있다.
- Flux는 state가 mutable하다.

|  | Flux | Redux |
| --- | --- | --- |
| Dispatcher | 단 하나의 Dispatcher | 없다. |
| Store | 여러 개의 Store, 갱신/보관 역할 | 하나의 Store, 보관 역할만 |
| State | mutable | immutable |

### Redux는 Flux와 이렇게 다르다.

- Store는 상태를 저장하는 것에만 책임이 있다.
    - Flux에서 Store는 상태를 갱신/저장하는 역할이었지만 Redux는 상태를 보관하는 역할만을 하고, 갱신은 다른 곳에서 진행한다. (Reducer)
- Reducer가 존재한다.
    - Reducer는 현재 상태와 Action을 인수로 받아 State의 복사본을 반환하는 순수 함수이다.
    - Redux는 상태를 변경할 수 없는 것으로 간주한다.
- Dispatcher가 존재하지 않는다.
    - Flux 패턴은 여러 Store가 존재하지만, Redux는 하나의 Store를 채택했기 때문에 여러 Store에 Action을 전달해주는 Dispatcher가 필요하지 않다.
    - Action은 Dispatcher에서 받지 않고 Reducer에서 Action이 분류되어 Store로 향한다.

### Reducer를 택한 장점

- 상태 업데이트 로직을 업데이트 할 때 hot reload를 사용할 수 있다.
    - Flux는 Store가 상태 업데이트 + 상태 보관의 역할을 하기 때문에 상태 업데이트 코드를 다시 작성하면 상태 또한 같이 reload된다. 즉, 저장된 상태 정보를 잃는다.
    - Redux는 Store에서 업데이트와 보관의 역할을 분리했기 때문에 상태 업데이트 로직인 Reducer를 reload해도 상태를 잃지 않을 수 있다.
- 시간 여행 디버깅을 할 수 있다.
    - 버그를 추적하기 위해 이전의 어플리케이션 상태로 돌아간다면 버그를 재현할 수 있다.
    - 시간 여행 디버깅을 위해서는 상태 객체의 모든 버전을 기록해두어야 한다. 자바스크립트에서는 객체를 수정하면 그의 참조들도 모두 변경되기 때문에 각각의 변화가 독립적인 객체로 존재해야 한다.
    - Redux에선 이를 해결하기 위해 Action이 전달되었을 때 현재 상태를 수정하는 것이 아니라 상태를 복사한 뒤 그 복사본을 수정한다.
- Redux는 Reducer를 여러 개를 만들어 도메인을 분리하므로 여러 개의 Store를 가질 필요가 없다.
    - 구독하고 있는 View가 상태가 업데이트 되었다는 알림을 받으면 상태는 완전히 업데이트된 상태임을 보장한다. 여러 개의 Store를 가질 경우 보장하기 쉽지 않다.
    - Store가 하나기 때문에 데이터에 접근하기가 쉽다.

### 단점

- 각 Reducer는 서로를 의존할 수 없고 완전히 고립되어 있다.
- Reducer를 불변성을 지키며 작성하기 어렵다.

## 참고

- [https://adjh54.tistory.com/49](https://adjh54.tistory.com/49)
- [https://basemenks.tistory.com/284](https://basemenks.tistory.com/284)
