# Spring5 리액티브 (Web flux)

WebFlux 설명에 앞서, 동기와 비동기 그리고 블럭과 넌블럭에 대해서 한 번 짚고 가자.\


![](https://velog.velcdn.com/images%2Frosa%2Fpost%2F123376b8-1d00-4351-88fa-3c6a7d78830b%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-12-11%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2010.00.01.png)

Blocking / Non-blocking

* Block : 호출된 함수가 자신이 할 일을 모두 마칠 때까지 제어권을 계속 가지고서 호출한 함수에게 바로 돌려주지 않는 것
* Non-block : 호출된 함수가 자신이 할 일을 채 마치지 않았더라고 바로 제어권을 건네주어 호출한 함수가 다른 일을 진행할 수 있도록 하는 것

Sync / Async

* Sync: 호출된 함수의 수행 결과 및 종료를 호출한 함수가 신경쓰는 것
* Async: 호출된 함수의 수행 결과 및 종료를 호출된 함수 혼자 직접 신경 쓰고 처리하는 것

출처 : [https://musma.github.io/2019/04/17/blocking-and-synchronous.html](https://musma.github.io/2019/04/17/blocking-and-synchronous.html)

***

#### Spring WebFlux <a href="#spring-webflux" id="spring-webflux"></a>

* Spring5 새로 추가된 모듈
* 클라이언트, 서버에서 reactive 스타일의 어플리케이션 개발을 돕는 모듈
* non-blocking 에 reactive stream 지원

**what is reactive?**

* I/O 이벤트에 반응하는 네트워크 컴포넌트, 마우스 이벤트에 반응하는 UI 컨트롤러 등과 같은 변화에 대한 반응을 중심으로 구축된 프로그래밍 모델
* non-blocking은 blocking과 다르게 작업이 완료되거나 데이터를 사용가능한 상태가 되면 알림에 반응하는 방식이 있으므로, reactive 하다고 한다.

**what is reactive programming?**

* 비동기 데이터 Stream으로 Non-Blocking 어플리케이션을 구현하는 프로그래밍
* Stream으로 프로그래밍 한다는 건\
  \- 함수형 처리가 가능해진다는 것
  * filter, map 등을 사용해 여러 형태로 편하게 사용 가능

**what is reactive streams?**

Non-bloking back presseure 를 이용한 비동기 데이터 처리의 표준.

**what is back pressure?**

푸시 방식: 기존 옵저버 패턴에서는 발행자가 구독자에게 밀어 넣는 방식으로 데이터 전달.\
풀 방식: 백 프레져는 구독자가 가져갈 수 있는 만큼 발행자가에게 요청하고, 발행자는 그 만큼만 전달. 전달되는 모든 데이터의 크기는 구독자가 결정하으로써 oom 발생 방지.

출처 :

* [https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1/](https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1/)
* [https://happymemoryies.tistory.com/24](https://happymemoryies.tistory.com/24)\
  참고 :
* [https://www.nurinamu.com/dev/2020/04/09/why-webflux-1/](https://www.nurinamu.com/dev/2020/04/09/why-webflux-1/)
* [https://www.reactive-streams.org/](https://www.reactive-streams.org/)

***

#### Spring MVC와 차이 <a href="#spring-mvc" id="spring-mvc"></a>

**Spring MVC 간략 설명**

* Java EE의 Servlet Spec 기반
* Blocking + Sync

**동작 흐름 (MVC & WebFlux)**

* MVC: request - Dispatcher Servlet - Handler Mapper - Controller - B/L - Controller - ViewResolver ...
* WebFlux : request - HttpHandler - WebHandler - Handler Mapper / Handler Adapter - Controller - B/L - 이하 같음

출처: [https://kimyhcj.tistory.com/343](https://kimyhcj.tistory.com/343) \[기억과 기록]

***

#### Spring WebFlux는 어떨 때 사용하는 게 좋은가 <a href="#spring-webflux" id="spring-webflux"></a>

WebFlux는 아래와 같은 용도로 사용하기를 추천한다고 합니다. (by 토비)

Non-blocking - Async reactive 개발에 사용하는 것이 좋음\
효율적으로 동작하는 고성능 웹어플리케이션 개발에 사용하는 것이 좋음\
서비스간 호출이 많은 마이크로 서비스 아키텍처에 적합

출처: [https://happymemoryies.tistory.com/24](https://happymemoryies.tistory.com/24) \[Happy Memory + ies (행복한 기억들의 기록)]
