# 코루틴이란? 쉽게!

\
**Kotlin**의 꽃이자, 많은 사람들이 **JAVA에서 Kotlin으로 넘어오는 큰이유**가 아닐까 싶습니다.

***

#### 1. 비동기 프로그래밍이 중요한 이유 <a href="#1" id="1"></a>

#### 1. 코루틴이란? <a href="#1" id="1"></a>

![](https://velog.velcdn.com/images%2Fvov3616%2Fpost%2Fb42b245b-a30d-4524-ac13-6aeca71622f5%2Fimage.png)

저희는 앱 개발 프로그래밍을

![](https://velog.velcdn.com/images%2Fvov3616%2Fpost%2F11212bbd-fe6d-467b-b0dc-507a11f071c1%2Fimage.png)

뭔가를 클릭할때마다 앱이 멈추니 사용자는 답답할것이다.\
이래서 코루틴은 동시성 프로그래밍을 지원한다.

> 함수를 중간에 빠져나왔다가, 다른 함수에 진입하고, 다시 원점으로 돌아와 멈추었던 부분부터 다시 시작하는 이 특성은 동시성 프로그래밍을 가능하게 한다.

코드로 생각해보자.\
[코루틴 개념 익히기](https://wooooooak.github.io/kotlin/2019/08/25/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0/)

![](https://velog.velcdn.com/images%2Fvov3616%2Fpost%2Fb9825b88-4045-4c3c-9649-44b405c77c89%2Fimage.png)

1. drawPersonToPaperA()를 호출.
2. startCouroutine을 만나고 drawHead에 진입했을때 suspend를 만나면 탈출
3. drawPersonToPaperA() 아래에 있는 drawCatToPaperB()를 실행.
4. B()의 drawHead를 만나고 탈출하여 A를 실행.
