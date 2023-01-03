# REST, REST API, RESTful 

## REST

### REST(Representational State Transfer)란?

자원을 **이름**(표현, representational)으로 구분하여 해당 자원의 정보(state)를 주고 받는 모든 것을 의미한다. 

- 웹의 장점을 최대한 활용할 수 있는 아키텍처 스타일
- client와 server 사이의 통신 방식 중 하나

### REST의 구체적인 개념은?

HTTP URI(Uniform Resource Identifier)를 통해 자원을 명시하고, 

HTTP Method(POST, GET, PUT, DELETE)를 통해 자원에 CRUD를 적용하는것 

- 자원 기반 구조 설계의 중심에 자원이 있고 이를 HTTP method를 통해 처리하도록 설계된 아키텍처

### REST가 필요한 이유는?

- 다양한 클라이언트가 존재하게 되었다. (모바일, 웹 등)
    - 멀티 플랫폼 지원을 위해 자원 활용 아키텍처를 세우고, 이용하는 방법을 모색한 결과 REST가 등장하였다.

### REST의 장점?

- HTTP protocol을 그대로 사용하므로 REST API를 사용하기 위한 별도의 인프라를 구축할 필요가 없다.
- 의도하는 바를 명확하게 파악할 수 있게 해준다.
- 서버와 클라이언트의 역할을 명확하게 분리한다.

### REST의 단점?

- 표준이 존재하지 않는다.
- HTTP 메서드가 한정적이다.
- 구형 브라우저가 아직 지원하지 못하는 부분이 존재한다.
    - PUT, DELETE

## REST API

REST 기반으로 API를 구현한 것 

### REST API 설계 규칙

- URI
    - 명사, 소문자 사용
    - 행위에 대한 동사 표현이 들어가면 안된다.
    - collection → 복수, document → 단수
    - 경로에서 변경되는 부분은 유일한 값으로 대체한다. (예시) id
    - 슬래시는 계층 관계를 나타내는데 사용한다.
    - 마지막 문자로 슬래시를 포함하지 않는다.
    - 불가피하게 경로가 길어진다면 (-) 하이픈을 사용한다.
    - 밑줄(_)은 사용하지 않는다.
    - 파일 확장자는 포함하지 않는다.
- HTTP method
    - URI에 HTTP method가 들어가면 안된다.
    - CRUD에 맞는 적절한 HTTP method가 사용되어야 한다.

## RESTful

REST 아키텍처를 구현하는 웹 서비스를 의미한다. 

- REST API를 제공하는 서비스는 RESTful 하다고 말할 수 있다.

### 목적

이해/사용이 쉬운 REST API를 만드는 것 

## 참고

- [https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html](https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html)
- [https://velog.io/@seokkitdo/Network-REST란-REST-API란-RESTful이란](https://velog.io/@seokkitdo/Network-REST%EB%9E%80-REST-API%EB%9E%80-RESTful%EC%9D%B4%EB%9E%80)
