# Spring MVC vs WebFlux

### 동기, 비동기, 블로킹, 논블로킹 <a href="#undefined" id="undefined"></a>

먼저 Spring MVC와 WebFlux의 차이점에 대해서 보기전에 동기/비동기, 블로킹/논블로킹에 대해서 알아보자.

### 동기(Synchronous) <a href="#synchronous" id="synchronous"></a>

필자는 동기라는 말을 처음봤을 때 무슨 뜻인지 이해조차 못했다.\
솔직히 블로킹이라는 말은 막혀있다? 막혀있는중이다? 등으로 해석하고 조금 이해가 가지만 동기라는 말은 무언가 명확하게 와 닿지 않았다.\
그래서 여러 글들을 보면서 아래와 같은 정의를 보고 간단하게 이해했다.

> 동기란 함수를 호출한 곳에서 응답을 받는 것

이 말은 호출과 응답이 동시에 이루어 지는 것을 의미하는 것이다.\
호출과 응답이 동시에 이루어 진다는 것 자체가 웃긴얘기일 수 있지만\
그냥 한마디로 비동기와 비교했을 때 처리결과를 받는 시점에 대한 차이가 있는 것이다.

### 비동기(Asynchronous) <a href="#asynchronous" id="asynchronous"></a>

비동기는 동기와 다르게 특정 호출시점과 처리결과에 대한 응답시점이 같지 않다는 것이다.\
예를들어 함수를 호출했을 때 바로 반환받고 처리결과에 대한 것은 추후에 처리가 완료되면 불특정 시점에 전달 받는 것을 의미한다.\
그래서 다시한번 말하면 동기(Synchronous)와 차이점은 결과에 대한 순서? 시점? 이라고 볼 수 있겠다.

### Blocking <a href="#blocking" id="blocking"></a>

> Blocking은 함수를 call 했을 때 응답을 받기 위해 멈춰있는 상태를 의미한다.

예를들어 우리가 fn hello()라는 함수를 만들었다고 가정했을 때 hello()라고 함수를 호출하면 함수의 return이 있고나서 함수가 종료되고 다음줄에 i++; 같은 코드를 실행하는 것이다.

### Non-blocking <a href="#non-blocking" id="non-blocking"></a>

> Non-blocking은 함수를 call 했을 때 blocking과 다르게 함수에 결과가 오기 전에 다음줄을 실행하는 상태를 말한다.

이것 또한 예를들면 hello()를 호출했어도 return이 되기 전에 다음줄의 i++; 같은 코드가 실행될 수 있는 상태를 의미하는 것이다.

간단하게나마 동기, 비동기, 블로킹, 논블로킹에 대해 설명했는데 사실 필자또한 적으면서 코드레벨에서 생각해봤을때 Synchronous & Blocking 조합 등이 같이 사용되는 것이 아닌가 라는 생각이 든다.

이런 개념을 미리 설명한 이유는 Spring MVC가 동기적으로 동작하고 WebFlux가 비동기적으로 동작하기 때문이다.\
이제 이 둘에 대해서 알아보자.

### Spring MVC <a href="#spring-mvc" id="spring-mvc"></a>

Spring MVC같은 경우는 우리가 Spring Framework와 Boot를 쓰면서 가장 보편적으로 쓰는 방식일 것이다. 필자 또한 이 방법을 많이 써왔다.\
Spring MVC의 경우 사용자의 요청이 들어왔을 때 그 때마다 Thread를 생성하여 처리한다.\
하지만 다수의 사용자 요청이 들어왔을 때 Thread를 계속 생성한다는 것은 Thread를 생성하는 등의 리소스가 굉장히 많이 들어간다.\
그래서 Spring MVC의 경우 어플리케이션이 실행되면서 Thread를 생성하면 Pool을 만들어 놓는다(Thread).\
요청이 들어오면 그 요청을 Queue에 쌓고 이것을 Thread Pool의 Thread들이 요청을 가져가 처리한다.\
이 방식의 단점이 있는데 엄청나게 많은 사용자가 동시에 요청을 보낼 경우 Pool Size를 초과하여 Queue에 계속 요청이 쌓이는 현상이 발생할 수 있다. 그래서 요청을 계속 처리하지 못하는 Thread Pool Hell이라는 현상이 발생할 수 있다는 것이다.

따라서 Spring MVC의 경우 개발하는 해당 시스템의 트래픽을 측정하여 Pool Size를 조정하는 것이 매우 중요하다고 할 수 있겠다.

### WebFlux <a href="#webflux" id="webflux"></a>

Spring WebFlux의 경우에는 계속 이슈가 되고 있는 Event driven 방식이고 비동기 논블로킹 방식이다.\
WebFlux는 Node.js처럼 이벤트 루프가 돌아서 요청이 발생할 경우 그것에 맞는 핸들러에게 처리를 위임하고 처리가 완료되면 callback 메소드 등을 통해 응답을 반환한다.\
그래서 이 방식의 경우 Spring MVC에 비해 사용자의 요청을 대량으로 받아낼 수 있다는 장점이 있는 것이다.\


![](https://velog.velcdn.com/images%2Fdyllis%2Fpost%2Fe0446d4f-9863-4718-b137-f22292b82ca3%2FTRM-10-FINAL.png)

하지만 무조건 WebFlux를 쓴다고 좋을까? 내 생각은 아니라고 말하고 싶다.\
요청을 처리하는 파이프라인의 요소들이 모두 논블로킹하게 동작해야만 의미가 있다.\
따라서 어떤 특정한 구간에서 블로킹이 발생하는 구간이 있다면 거기서부터 Thread Pool Hell같은 문제들이 발생하는 것이다.\
그래서 우리가 개발하는 어플리케이션의 지표를 측정해서 WebFlux로 효율을 볼 수 있는 구간에 도달했을 때 적용하는 것이 맞다고 생각이 든다.\


![](https://velog.velcdn.com/images%2Fdyllis%2Fpost%2Fb5dc5dab-1f82-41bf-9066-4797f25e72b6%2FBoot1VsBoot2.png)

\
Boot1이 Spring MVC로 개발된 것이고 Boot2가 WebFlux이다. 이처럼 두 어플리케이션의 성능이 차이가 나는 구간이 있다. 이때 WebFlux를 적용해야 효율을 볼 수 있지 않을까 생각한다.\
그래서 만약 Spring MVC에서 WebFlux로 전환을 생각한다면 이런 지표를 보고 전환해서 효율을 볼 수 있으면 전환하는 것이 맞을 것이라 생각한다.
