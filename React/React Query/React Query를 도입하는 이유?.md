### React Query를 도입하는 이유?

도입하는 가장 큰 이유는 서버 데이터와 클라이언트 데이터를 분리하겠다는 것이다. 
서버 데이터를 전역 store에 저장하는 이유는 두 가지가 있다고 생각한다.

1. 서버에서 받아온 데이터를 여러 관계없는 컴포넌트에서 사용해야할 때 
2. API 요청 감소를 위한 캐싱용도 

만약 전역 상태 라이브러리로 redux를 사용했고, redux store에 서버 데이터를 저장할 경우 

- 데이터 동기화를 위한 코드나, 데이터 패칭 상태에 대한 코드를 작성해주어야 해서 로직이 과도하게 커진다.
- 서버 데이터와 클라이언트 데이터 구분이 모호해진다. (서버 응답이 전역 store에 저장되지 않아도 되는 경우에도 코드의 통일성을 위해 action과 reducer를 작성하게 된다.)

여기서 React Query를 사용하면 문제를 해결할 수 있다. 

- 캐싱을 통해 API 요청을 감소시킬 수 있다.
- 데이터가 stale하다고 판단되면, 직접 코드를 작성하지 않아도 다시 데이터 패칭을 해온다.
- 서버 데이터와 클라이언트 데이터를 분리할 수 있다.
- 에러 핸들링 관련 코드와 데이터 패칭 상태 관련 코드를 줄일 수 있다.
- 로딩 상태나, 에러 상태를 리액트의 suspense를 활용하여 선언적으로 표시해줄 수 있다.
