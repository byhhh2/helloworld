# ECMAScript

## 역사

- 1990년대 Netscape Navigator 브라우저에 동적인 부분을 더하기 위한 목적을 Brianden Eich에 의해 개발되었다.
- Netscape는 언어의 표준화, 명세화를 위해 ECMA International에 제출했다.
    - 이름 합의 끝에 ECMAScript, ECMA-262라는 이름으로 탄생했다.
- Javascript는 ECMA-262를 만족하는 구현체를 가르킨다.

![Untitled (9)](https://user-images.githubusercontent.com/52737532/201579499-de737008-d7ae-44d9-bdd3-b9591f7024c3.jpeg)

## ECMAScript

- 자바스크립트 표준사양인 ECMA-262
- 핵심 문법을 규정한다.
- 각 브라우저 제조사는 ECMAScript 사양을 준수해서 자바스크립트 엔진을 구현한다.

## Javascript

- 프로그래밍 언어로서 기본 뼈대를 이루는 ECMAScript + 클라이언트 사이드 Web API + DOM + BOM…
    - 클라이언트 사이드 Web API는 W3C에서 별도의 사양으로 관리하고 있다.

## TC39

- ECMA International의 여러 기술 위원회 중 TC39라는 위원회가 ECMA-262 명세를 관리한다.
- TC39는 Mozilla, Google, Apple.. 등 메이저 브라우저 벤더를 비롯해 여러 단체로 이루어져 있다.
- TC39는 정기적으로 만나 회의를 진행하는데, 회의록은 모두 웹상에 공개된다.

## TC39 프로세스

- 0단계부터 4단계까지, 아직 표준에 편입되지도, 명시적으로 거절되지도 않은 제안을 통틀어 proposal이라 칭한다.
- 각 단계로의 승급을 위한 명시적인 조건이 존재한다.

### stage 0 : strawman (허수아비)

- 라이센스 관련 조항에 동의하고 TC39 컨트리뷰터로 등록한 누구라도 proposal을 낼 수 있다.
- 회의의 안건으로 상정되고, 0단계 문서에 등재되면 0단계 제안이 된다.

### stage 1 : proposal (제안)

- 1단계에 들어오기 위해서는 챔피언(champion)을 구해야한다.
    - 챔피언 : 해당 제안을 책임지고 다음 단계로 끌고 나아갈 TC39 구성원
- 풀고자 하는 문제, 잠재적 장애물을 제시해야한다.
- 구현상으로는 폴리필, 데모 등을 필요로 한다.

### stage 2 : draft (초안)

- ECMAScript 표준의 형식 언어(formal description)로 작성된 형식적인 서술 초안이 필요하다
    - 명세가 표준에 편입될 경우 사용할 명세의 초기 버전이다.
- 앞으로 해야할 일을 표시하는 등 일부 불완전한 명세가 허용된다. 또, 실험적인 구현이 요구된다.
- 2단계까지 올라왔다는 것은 위원회가 이를 표준에 포함시키고자 하는 의지가 있다고 해석할 수 있다.

### stage 3 : candidate (후보)

- 대부분 완성에 가깝고, 피드백을 받아보는 일만 남은 상태
- 초안과는 다르게 문법, 동작, API까지 모든 부분이 기술되어 있도록 마무리된 명세가 필요하다.
- 구현상 심각한 문제가 발결되지 않는 이상 변경이 허용되지 않는다.

### stage 4 : finished (완료)

- 제안이 수락되고, 다음 표준에 포함되어 발표되기만을 기다리는 단계
- proposal이 ECMA-262 단위 테스트 슈트인 Test262에 관련 테스트가 작성되고, 까다로운 추가 조건을 모두 만족하면 4단계로 올라올 수 있다.
- 4단계까지 올라온 proposal은 별다른 이변이 없는 이상 다가오는 새 표준에 포함되어 발표된다.
    - 매년 6월에 ECMAScript 표준이 발표된다.

## 참고

- [https://ahnheejong.name/articles/ecmascript-tc39/](https://ahnheejong.name/articles/ecmascript-tc39/)
