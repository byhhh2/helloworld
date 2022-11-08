# 미리 좀 가져와줄래 ?

**요청 우선순위 조정**

> **<link> 태그의 rel 속성**
> 
> - 현재 문서와 외부 리소스 사이의 연관 관계를 명시한다.
> - rel 속성은 반드시 명시되어야 하는 필수 속성이다.

## Preload

> 💬 미리 좀 가져와줄래?
> 
> 
> 우선순위 : `high`
> 
> 필수 속성 : rel, as, crossorigin (font일 경우) 
> 

```html
<link rel="preload" href="<%= PUBLIC_URL %>/static/font.woff2" as="font" type="font/woff2" crossorigin />

<!-- as : 리소스의 유형 명시 -->
```

현재 페이지에서 바로 필요한 리소스로, 빠르게 가져와야 한다는 것을 브라우저에게 알린다.   
브라우저가 as 속성과 대상의 우선순위에 따라 현재 탐색에 사용할 해당 리소스를 미리 가져와 캐시하도록 명시   

### 특징

- fetch는 하지만, 바로 excute 하진 않는다. (스크립트를 미리 받아와도 실행은 나중에 한다.)
- onload 이후 3초 이내에 preload한 리소스를 사용하지 않을 경우 크롬에서 경고 로그 발생
- 받아온 뒤에 브라우저에 캐시한다.

### 사용처

- 폰트
- 초기 렌더링에 반드시 필요한 리소스 (critical rendering path)

> **<link> 태그의 as 속성**
> 
> 
> 
> - rel = preload 또는 rel = prefetch 특성을 지정했을 때만 사용
> - rel = preload는 as 특성을 사용해 요청 우선 순위를 매긴다.

> **crossorigin**
> 
> 
> [why do we need crossorigin when preloading font](https://stackoverflow.com/questions/70183153/why-do-we-need-the-crossorigin-attribute-when-preloading-font-files)
> 
> - 리소스를 가져올 때 CORS를 사용해야 하는지 나타내는 특성
> - font는 crossorigin 속성을 포함하지 않았을 때 font를 두번 중복해서 가져온다.
>     - 속성이 없으면 브라우저에서 미리 로드된 글꼴을 무시한다.
>     - fetch가 교차 출처가 아닌 경우에도 crossorigin 속성을 설정해야한다.
>     - 왜? 레거시 리소스 유형을 지원해야 하는 최신 기능이기 때문에

> **Critical Rendering Path**
> 
> 
> 
> 브라우저가 HTML, CSS, Javascript를 화면에 픽셀로 변화하는 일련의 단계
> 
> - 이를 최적화하는 것은 렌더링 성능을 향상시킨다.
> - DOM, CSSOM, render tree, Layout을 포함한다.

## Prefetch

> 💬 나중에 쓰일 것 같은데, 미리 좀 가져와줄래 ?
> 
> 
> 우선순위 : `low`
> 

브라우저에게 미래에 필요할 수 있는 페이지, 리소스(script, css) 등을 미리 다운로드 받아두라고 알려주는 것 

### 특징

- 현재 페이지가 다 로드된 이후 후순위로 받아서 prefetch cache에 넣는다.
- 현재 페이지 로드 시간에는 영향을 미치지 않지만, 다음 navigation의 FCP나 TTI에 영향을 미친다.
    - 후속 페이지 로드 시간을 개선할 수 있다.
    - ex) 사용자 인증 중에 다음 페이지를 미리 로드 한다.
    - **FCP** : 최초 컨텐츠풀 페인트 (First Contentful Paint), 페이지가 로드되기 시작한 시점부터 페이지 컨텐츠의 일부가 화면에 렌더링 될 때까지의 시간
    - **TTI** : 상호 작용까지의 시간 (Time To Interactive), 페이지가 로드되기 시작한 시점부터, 사용자 입력에 안정적으로 응답할 수 있는 시점까지의 시간
- 초기 렌더링에선 필요없는 리소스를 불러올 때 사용

### 사용처

- 초기 렌더링에서 필요없는 리소스
- 같은 어플리케이션에서 다른 페이지에 사용되는 JS 번들
- 현재 페이지에서 쓰이더라도 초기 렌더링에는 필요없는 리소스인 모달

## Preconnect

> 💬 연결만 미리 해놓을까 ?
> 
> 
> 우선순위 : `low`
> 

브라우저에서 서버와 연결만 미리 맺어두고 있어 달라고 알려주는 것 

<img width="634" alt="스크린샷 2022-11-09 오전 1 29 12" src="https://user-images.githubusercontent.com/52737532/200630389-576d1382-7da6-4aa9-adce-dd27cc99a93c.png">

리소스를 가져오기 위해서는 연결을 하고 다운로드하는 과정이 필요하다.   
(연결 : 도메인 이름을 찾아 IP 주소 확인 → 서버에 대해 연결 설정 → 보안 위해 연결 암호화)   

### 특징

- 개별 요청당 100~500ms 정도 로드 시간을 절약할 수 있다.

### 사용처

- 외부 도메인 리소스 (폰트)

# 이건 바로는 필요 없어

자바스크립트 : **파서 차단 리소스** (parser blocking resource) 

![Untitled (11)](https://user-images.githubusercontent.com/52737532/200630569-26d19c9a-0d48-4f8b-91a5-cb58d5a5c6d0.png)

![Untitled (12)](https://user-images.githubusercontent.com/52737532/200630589-85ef50ed-d6fb-4770-a9fa-5c565a2820c1.png)


## defer

- HTML parsing에 script fetch, execution 둘 다 영향가지 않게 한다.
    - fetch는 parsing 도중에 진행된다. (parsing은 멈추지 않는다.)
- HTML parsing이 끝날 때까지 스크립트 실행을 지연한다.
- head에 script가 있어도 전체 DOM이 구성된 이후에 실행

```html
defer script fetch -> DOMContentLoaded -> defer script execute 
```

> `load`는 모든 리소스(CSS, images..)가 다운로드된 다음에 호출되고,
> 
> 
> `DOMContentLoaded`는 HTML document를 전부 읽고 DOM트리를 완성하는 즉시 이벤트가 호출된다.
> 

## async

- HTML parsing에 fetch만 영향가지 않는다. (excution은 영향간다.)
- fetching이 완료되면 HTML parsing 중간에 스크립트가 실행될 수 있다.
- 광고 등 문서 내용이나 앱 자체와는 관련 없이 독립적으로 동작하는 스크립트에 사용한다.

## 참고

- [tcpschool - link.rel](http://www.tcpschool.com/html-tag-attrs/link-rel)
- 우아한테크코스 LMS
- 밧드의 preload, prefetch, preconnection
- [critical rendering path](https://developer.mozilla.org/ko/docs/Web/Performance/Critical_rendering_path)
- [as, crossorigin](https://developer.mozilla.org/ko/docs/Web/HTML/Element/link#attr-as)
- [fcp](https://web.dev/i18n/ko/fcp/)
- [https://web.dev/preconnect-and-dns-prefetch/](https://web.dev/preconnect-and-dns-prefetch/)
- [https://web.dev/preconnect-and-dns-prefetch/](https://web.dev/preconnect-and-dns-prefetch/)
- [DOMContentLoaded, load](https://yelee.tistory.com/13)
