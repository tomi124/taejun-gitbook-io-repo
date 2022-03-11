---
description: 코틀린 동시성 프로그래밍 1장을 요약한 내용입니다.
---

# 1장 Hello, Concurrent World!



## 프로세스, 스레드, 코루틴

애플리케이션을 시작할 때 운영체제는 프로세스를 생성하고 여기에 스레드를 연결한 다음, 메인 스레드(main thread)로 알려진 해당 스레드를 시작한다.

### 프로세스

프로세스는 실행 중인 애플리케이션의 인스턴스다. 애플리케이션은 여러 프로세스로 구성될 수 있다.

### 스레드

실행 스레드는 프로세스가 실행할 일련의 명령을 포함한다. 스레드가 끝나면 프로세스의 다른 스레드와 상관없이 프로세스가 종료된다.

```kotlin
fun main(args: Array<String>) {
		doWork()
}
```

기본적인 애플리케이션이 실행되면 main() 함수의 명령 집합이 포함된 메인 스레드가 생성된다. doWork()은 메인 스레드에서 실행되므로 doWork()이 종료되면 애플리케이션의 실행이 종료된다.

스레드 안에서 명령은 한 번에 나라씩 실행돼 스레드가 블록(block)되면 블록이 끝날 때까지 같은 스레드에서 다른 명령을 실행할 수 없다. 그러나 많은 스레드가 같은 프로세스에서 생성될 수 있으며 서로 통신할 수 있다.

코틀린이 동시성을 구현한 방식을 보면 여러분이 직접 스레드를 시작하거나 중지할 필요가 없다는 것을 알게 된다. 한두 줄의 코드로 코틀린이 특정 스레드나 스레드 풀을 생성해서 코루틴을 실행하도록 지시하기만 하면 된다. 스레드와 관련된 나머지 처리는 프레임워크에 의해 수행된다.

### 코루틴

코틀린 문서에서는 코루틴을 경량 스레드라고도 한다. 대부분의 스레드와 마찬가지로 코루틴이 프로세서가 실행할 명령어 집합의 실행을 정의하기 때문이다. 또한 코루틴은 스레드와 비슷한 라이프 사이클을 갖고 있다.

코루틴은 스레드 안에서 실행된다. 스레드 하나에 많은 코루틴이 있을 수 있지만 주어진 시간에 하나의 스레드에서 하나의 명령만이 실행될 수 있다. 즉 같은 스레드에 10개의 코루틴이 있다면 해당 시점에는 하나의 코루틴만 실행된다. 스레드와 코루틴의 가장 큰 차이점은 코루틴이 빠르고 적은 비용으로 생성할 수 있다는 것이다. 수천 개의 코루틴도 쉽게 생성할 수 있으며, 수천 개의 스레드를 생성하는 것보다 빠르고 자원도 훨씬 적게 사용한다.

다음 함수는 파라미터 amount에 지정된 수만큼 코루틴을 생성해 각 코루틴을 1초 간 지연시킨 후 모든 코루틴이 종료될 때까지 기다렸다가 반환한다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    println("${Thread.activeCount()} threads active at the start")
    val time = measureTimeMillis {
        createCoroutines(3)
    }
    println("${Thread.activeCount()} threads active at end")
    println("Took $time ms")
}

suspend fun createCoroutines(amount: Int) {
    val jobs = ArrayList<Job>()
    for (i in 1..amount) {
        jobs += GlobalScope.launch {
            delay(1000)
        }
    }
    jobs.forEach {
        it.join()
    }
}
```

테스트 환경에서 amount를 10,000으로 실행할 때 약 1,160ms가 걸리는데 반해 100,000으로 실행하는 데 1,649ms가 소요됐다. 코틀린은 고정된 크기의 스레드 풀을 사용하고 코루틴을 스레드들에 배포하기 때문에 실행 시간이 매우 적게 증가한다. 따라서 수천 개의 코루틴을 추가하는 것은 거의 영향이 없다.

코루틴이 특정 스레드 안에서 실행되더라도 스레드와 묶이지 않는다는 점을 이해해야 한다. 코루틴의 일부를 특정 스레드에서 실행하고, 실행을 중지한 다음 나중에 다른 스레드에서 계속 실행하는 것이 가능하다.

```kotlin
suspend fun createCoroutines(amount: Int) {
    val jobs = ArrayList<Job>()
    for (i in 1..amount) {
        jobs += GlobalScope.launch {
            println("Started $i in ${Thread.currentThread().name}")
            delay(1000)
            println("Finished $i in ${Thread.currentThread().name}")
        }
    }
    jobs.forEach {
        it.join()
    }
}
```

이와 같이 다른 스레드서 다시 시작하는 경우가 많음을 알게 될 것이다.

스레드는 한 번에 하나의 코루틴만 실행할 수 있기 때문에 프레임워크가 필요에 따라 코루틴을 스레드들 사이에 옮기는 역할을 한다.

### 내용 정리

스레드를 블록한다는 것은 그 스레드에서 코드의 실행을 중지한다는 의미인데, 사용자와 상호작용하는 스레드는 블록되지 않아야 한다.

동시성은 애플리케이션이 동시에 한 개 이상의 스레드에서 실행될 때 발생한다.

### 동시성에 대해

올바른 동시성 코드는 결정론적인 결과를 갖지만 실행 순서에서는 약간의 가변성을 허용하는 코드다. 그러려면 서로 다른 부분이 어느 정도 독립성이 있어야 하며 약간의 조정도 필요하다. 동시성을 이해하는 가장 좋은 방법은 순차적인 코드를 동시성 코드와 비교하는 것이다.

```kotlin
fun getProfile(id: Int): Profile {
    val basicUserInfo = getUserInfo(id)
    val contactInfo = getContactInfo(id)

    return createProfile(basicUserInfo, contactInfo)
}
```

여기서 중요한 것은 사용자 정보가 반환되기 전까지 연락처 정보를 요청하지 않는다는 것이다. 이것이 순차 코드의 장점이다. 정확한 실행 순서를 쉽게 알 수 있어서 예측하지 못한 일이 벌어지지는 않을 것이다. 그러나 순차 코드에는 두 가지 큰 문제점이 있다.

* 동시성 코드에 비해 성능이 저하될 수 있다.
* 코드가 실행되는 하드웨어를 제대로 활용하지 못할 수 있다.

getUserInfo가 5초가 걸리고 getContactInfo가 5초가 걸린다면 getProfile은 항상 10초 이상 걸릴 것이다.

getProfile의 동시성 구현에 관래 살펴보자.

```kotlin
suspend fun getProfile(id: Int): Profile {
    val basicUserInfo = asyncGetUserInfo(id)
    val contactInfo = asyncGetContactInfo(id)

    return createProfile(basicUserInfo.await(), contactInfo.await())
}
```

asyncGetUserInfo()와 asyncGetContactInfo()는 서로 다른 스레드에서 실행되도록 작성됐기 때문에 동시성이라고 한다.

getProfile의 동시성 구현 버전은 순차적 구현보다 두 배 빠르게 수행될 수 있지만 실행할 때 약간의 가변성이 있다. 그것이 createProfile()을 호출할 때 두개의 wait() 호출이 있는 이유다. asyncGetUserInfo()와 asyncGetContactInfo()가 모두 완료될 때까지 getProfile()의 실행을 일시 중단한다는 것이다.

그것이 동시성의 까다로운 부분이다. 코드의 준독립적인(semi-independent)부분이 완성되는 순서에 관계없이 결과가 결정적이어야 함을 보장해야 한다.

### 동시성은 병렬성이 아니다

흔히 동시성과 병렬성을 혼동하곤 한다. 두 개의 코드가 동시에 실행된다는 점에서 둘 다 비슷해 보이긴 한다. 둘의 차이점은 같은 프로세스 안에서 서로 다른 명령 집합의 타임라인이 겹칠 때 동시성이 발생한다는 점이다. 동시성은 정확히 같은 시점에 실행되는지 여부와는 상관이 없다. 만약 위의 코드를 단일 코어에서 발생한다면 동시성이지만 병렬은 아니다. 단일 처리 장치는 X와 Y 스레드 사이에 교차 배치되며, 두 개의 전체 일정이 겹치지만 지정된 시점에 둘 중 하나만 실행되고 있다.

반면에 병렬 실행은 두 스레드가 정확히 같은 시점에 실행될 때만 발생한다. 두 개의 코어가 있는 컴퓨터에서 getProfile()이 실행되고 있는 경우 코어 하나는 asyncGetUserInfo()의 명령을 실행하고 다른 하나의 코어에서 asyncGetContactInfo()의 명령을 실행한다. 요약하면 다음과 같다.

* 동시성은 두 개 이상의 알고리즘의 실행 시간이 겹쳐질 때 발생한다. 만약 단일 코어에서 실행되면 병렬이 아니라 동시에 실행되는데, 단일 코어가 서로 다른 스레드의 인스트럭션을 교차배치해서 스레드들의 실행을 효율적으로 겹쳐서 실행한다.
* 병렬은 두 개의 알고리즘이 정확히 같은 시점에 실행될 때 발생한다. 이것이 가능하려면 2개 이상의 코어와 2개 이상의 스레드가 있어야 각 코어가 동시에 스레드의 인스트럭션을 실행할 수 있다.

## CPU 바운드와 I/O 바운드

### CPU 바운드

CPU만 완료하면 되는 작업을 중심으로 구현되는 알고리즘이 많다. 알고리즘의 성능은 실행 중인 CPU의 성능에 좌우되며 CPU만 업그레이드해도 성능이 향상된다.

단어를 가져와서 좌우가 같은 단어인지를 판별하는 간단한 알고리즘을 살펴보자

```kotlin
fun main(args: Array<String>) {
    filterPalindromes(words).forEach {
        println(it)
    }
}

fun filterPalindromes(words: List<String>) : List<String> {
    return words.filter { isPalindrome(it) }
}

fun isPalindrome(word: String) : Boolean {
    val lcWord = word.toLowerCase()
    return lcWord == lcWord.reversed()
}
```

수십 만 개의 단어를 보내도록 코드를 바꾸면 filterPalindromes()는 더 오래 걸릴 것이다. 코드를 더 빠른 CPU에서 실행하면 코드의 변경 없이도 성능이 향상된다.

### I/O 바운드

I/O 바운드는 입출력 장치에 의존하는 알고리즘이다. 실행 시간은 입출력 장치의 속도에 따라 달라지는데, 예컨대 문서를 읽어서 문서의 각 단어를 filterPalindromes()에 전달해 좌우가 같은 단어를 출력하는 알고리즘이 I/O 바운드다. 네트워킹이나 컴퓨터 주변기기로부터의 입력을 받는 작업들도 I/O 작업이다.

### CPU 바운드 알고리즘에서의 동시성과 병렬성

CPU 바운드 알고리즘의 경우 다중 코어에서 병렬성을 활용하면 성능을 향상시킬 수 있지만 단일 코어에서 동시성을 구현하면 성능이 저하되기도 한다.

#### 단일 코어에서 실행

단일 코어에서 실행된다면 하나의 코어가 3개의 스레드 사이에서 교차 배치되며 매번 일정량의 단어를 필터링하고 다음 스레드로 전환된다. 전환 프로세스를 컨텍스트 스위칭이라고 한다.

컨텍스트 스위칭은 현재 스레드의 상태를 저장한 후 다음 스레드의 상태를 적재해야 하기 때문에 전체 프로세스에 오버헤드가 발생한다. 순차적 구현에서는 단일 코어가 모든 작업을 수행하지만 컨텍스트 스위칭이 발생하지 않기 때문이다.

#### 병렬 실행

병렬 실행의 경우 각 스레드가 하나의 전용 코어에서 실행된다고 가정하면 isPalindrome()의 실행은 순차적 실행의 약 3분의 1이 될 것이다.

CPU 바운드 알고리즘을 위해서는 현재 사용 중인 장치의 코어 수를 기준으로 적절한 스레드 수를 생성하도록 고려해야만 한다.

### I/O 바운드 알고리즘에서의 동시성 대 병렬성

I/O 바운드 알고리즘은 순차적인 알고리즘보다 동시성 구현에서 항상 더 나은 성능을 발휘할 것으로 예상돼 I/O 작업은 늘 동시성으로 실행하는 편이 좋다.

## 동시성이 어려운 이유

어떤 언어들은 너무 번거롭게 동시성 코드를 만들기도 하고, 융통성 없이 만들어 사용성을 떨어뜨리는 언어도 있다. 그러나 코틀린은 우리가 만든 동시성 코드를 동기화하고 통신할 수 있게 만들기 때문에 실행 흐름이 바뀌어도 애플리케이션의 작동에는 영향이 없다.

### 레이스 컨디션

동시성 코드를 작성할 때 가장 흔한 오류인 레이스 컨디션은 코드를 동시성으로 작성했지만 순차적 코드처럼 동작할 것이라고 예상할 때 발생한다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    asyncGetUserInfo(1)
    // Do some other operations
    delay(1000)

    println("User ${user.id} is ${user.name}")
}

fun asyncGetUserInfo(id: Int) = GlobalScope.async {
    delay(1100)
    user = UserInfo(id = id, name = "Susan", lastName = "Calvin")
}
```

위 코드를 실행하면 user에 정보를 출력하는 동안 초기화되지 않아서 main()가 중단된다. 레이스 컨디션을 고치려면 정보에 접근하려고 하기 전에 정보를 얻을 때까지 명시적으로 기다려야만 한다.

### 원자성 위반

원자성 작업이란 작업이 사용하는 데이터를 간섭 없이 접근할 수 있음을 말한다. 단일 스레드 애플리케이션에서는 모든 코드가 순차적으로 실행되기 때문에 모든 작업이 모두 원자일 것이다. 스레드가 하나만 실행되므로 간섭이 있을 수 없다.

수정이 겹칠 수 있다는 것은 데이터 손실이 발생할 수 있다는 뜻인데, 가령 코루틴이 다른 코루틴이 수정하고 있는 데이터를 바꿀 수 있다는 것이다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val workerA = asyncIncrement(2000)
    val workerB = asyncIncrement(100)

    workerA.await()
    workerB.await()

    print("counter [$counter]")
}

var counter = 0

fun asyncIncrement(by: Int) = GlobalScope.async {
    for (i in 0 until by) {
        counter++
    }
}
```

main()을 실행하면 대부분 counter \[2100]를 출력하지만, 꽤 많은 실행에서 2,100보다 적은 값을 인쇄한다는 것을 의미한다.

### 교착 상태

동시성 코드가 올바르게 동기화되려면 다른 스레드에서 작업이 완료되는 동안 실행을 일시 중단하거나 차단할 필요가 있다.

```kotlin
// This will never complete execution.
fun main(args: Array<String>) = runBlocking {
    jobA = launch {
        delay(1000)
        // wait for JobB to finish
        jobB.join()
    }

    jobB = launch {
        // wait for JobA to finish
        jobA.join()
    }

    // wait for JobA to finish
    jobA.join()
    println("Finished")
}
```

### 라이브 락

라이브 락(Livelocks)은 애플리케이션이 올바르게 실행을 계속할 수 없을 때 발생하는 교착상태와 유사하다. 라이브 락이 진행될 때 애플리케이션의 상태는 지속적으로 변하지만 애플리케이션이 정상 실행으로 돌아오지 못하게 하는 방향으로 상태가 변한다는 점이 다르다.

교착 상태를 복구하도록 설계된 알고리즘에서 라이브 락이 발생하는 경우가 많다. 교착 상태에서 복구하려는 시도가 라이브 락을 만들어 낼 수도 있다.

## 코틀린에서의 동시성

### 넌 블로킹

스레드는 무겁고 생성하는 데 비용이 많이 들며 제한된 수의 스레드만 생성할 수 있다. 코틀린은 채널(channels), 액터(actors), 상호 배제(mutual exclusions)와 같은 훌륭한 기본형(primitives)도 제공해 스레드를 블록하지 않고 동시성 코드를 효과적으로 통신하고 동기화하는 메커니즘을 제공한다.

### 명시적인 선언

동시성은 연산이 동시에 실행돼야 하는 시점을 명시적으로 만드는 것이 중요하다. 일시 중단 가능한 연산은 기본적으로 순차적으로 실행된다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val time = measureTimeMillis {
        val name = getName()
        val lastName = getLastName()

        println("Hello, ${name} ${lastName}")
    }

    println("Execution took $time ms")
}

suspend fun getName(): String {
    delay(1000)
    return "Susan"
}

suspend fun getLastName(): String {
    delay(1000)
    return "Calvin"
}
```

코드에서 main()은 현재 스레드에서 일시 중단 가능한 연산 getName()과 getLastName()을 순차적으로 실행한다.

그러나 getLastName()과 getName() 간에 의존성이 없기 때문에 동시에 수행하는 폇이 더 낫다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val time = measureTimeMillis {
        val name = async { getName() }
        val lastName = async { getLastName() }

        println("Hello, ${name.await()} ${lastName.await()}")
    }

    println("Execution took $time ms")
}
```

async { ... }를 호출해 두 함수를 동시에 실행해야 하며 await()를 호출해 두 연산에 모두 결과가 나타날 때까지 main()이 일시 중단되도록 요청한다.

### 가독성

코틀린의 동시성 코드는 순차적 코드만큼 읽기 쉽다. suspend 메소드는 백그라운드 스레드에서 실행될 두 메소드를 호출하고 정보를 처리하기 전에 완료를 기다린다.

### 기본형 활용

스레드를 만들고 관리하는 것은 여러 프로그래밍 언어에서 동시성 코드를 작성할 때 가장 어려운 부분 중 하나다. 언제 스레드를 만들 것인가를 아는 것 못지 않게 얼마나 많은 스레드를 만드는지를 아는 것도 중요하다. 또한 I/O 작업 전용 스레드와 CPU 바운드 작업을 처리하는 스레드가 있어야 하는데, 스레드를 통신/동기화하는 것은 그 자체로 어려운 일이다.

코틀린은 동시성 코드를 쉽게 구현할 수 있는 고급 함수와 기본형을 제공한다.

* 스레드는 스레드 이름을 파라미터로 하는 newSingleThreadContext()를 호출하면 생성된다. 일단 생성되면 필요한 만큼 많은 코루틴을 수행하는 데 사용할 수 있다.
* 스레드 풀은 크기와 이름을 파라미터로 하는 newFixedThreadPoolContext()를 호출하면 쉽게 생성할 수 있다.
* CommonPool은 CPU 바운드 작업에 최적인 스레드 풀이다. 최대 크기는 시스템의 코어에서 1을 뺀 값이다.
* 코루틴을 다른 스레드로 이동시키는 역할은 런타임이 담당한다.
* 태널, 뮤텍스 및 스레드 한정과 같은 코루틴의 통신과 동기화를 위해 필요한 많은 기본형과 기술이 제공된다.

### 코틀린 동시성 관련 개념과 용어

#### 일시 중단 연산

일시 중단 연산은 해당 스레드를 차단하지 않고 실행을 인시 중지할수 있는 연산이다.

#### 일시 중단 함수

일시 중단 함수는 함수 형식의 일시 중단 연산이다.

```kotlin
suspend fun greetAfter(name: String, delayMillis: Long) {
		delay(delayMillis)
		println("Hello, $name")
}
```

앞의 예제에서 greetAfter()의 실행은 delay()가 호출될 때 일시 중단된다. delay()는 자체가 일시 중단 함수이며, 주어진 시간 동안 실행을 일시 중단한다. delay()가 완료되면 greetAfter()가 실행을 정상적으로 다시 시작한다. greetAfter()가 일시 중지된 동안 실행 스레드가 다른 연산을 수행하는 데 사용될 수 있다.

### 람다 일시 중단

일반적인 람다와 마찬가지로, 일시 중단 람다는 익명의 로컬 함수다. 일시 중단 람다는 다른 일시 중단 함수를 호출함으로써 자신의 실행을 중단할 수 있다는 점에서 보통의 람다와 차이가 있다.

### 코루틴 디스패처

코루틴을 시작하거나 재개할 스레드를 결정하기 위해 코루틴 디스패터가 사용된다. 모든 코루틴 디스패터는 CoroutineDispatcher 인터페이스를 구현해야 한다.

* DefaultDispatcher: 현재는 CommonPool과 같다. 앞으로 바뀔 수 있다.
* CommonPool: 공유된 백그라운드 스레드 풀에서 코루틴을 실행하고 다시 시작한다. 기본 크기는 CPU 바운드 작업에서 사용하기에 적합하다.
* Unconfined: 현재 스레드에서 코루틴을 시작하지만 어떤 스레드에서도 코루틴이 다시 재개될 수 있다. 디스패터에서는 스레드 정책을 사용하지 않는다.

디스패처와 함께 필요에 따라 Pool 또는 스레드를 정의하는 데 사용할 수 있는 몇 가지 빌더가 있다.

* newSingleThreadContext() : 단일 스레드로 디스패처를 생성한다. 여기에서 실행되는 코루틴은 항상 같은 스레드에서 시작되고 재개된다.
* newFixedThreadPoolContext() : 지정된 크기의 스레드 풀이 있는 디스패처를 만든다. 런타임은 디스패처에서 실행된 코루틴을 시작하고 재개할 스레드를 결정한다.

### 코루틴 빌더

코루틴 빌더는 일시 중단 람다를 받아 그것을 실행시키는 코루틴을 생성하는 함수다.

* async() : 결과가 예상되는 코루틴을 시작하는 데 사용된다. async()는 코루틴 내부에서 일어나는 모든 예외를 캡처해서 결과에 넣기 때문에 조심해서 사용해야 한다.
* launch() : 결과를 반환하지 않는 코루틴을 시작한다. 자체 혹은 자식 코루틴의 실행을 취소하기 위해 사용할 수 있는 Job을 반환한다.
* runBlocking() : 블로킹 코드를 일시 중지 가능한 코드로 연결하기 위해 작성됐다. 보통 main() 메소드와 유닛 테스트에서 사용된다.

```kotlin
val result = GlobalScope.async(Dispatchers.Unconfined) { // 디스패처를 수동으로 지정할 수 있다. 
		isPalindrome(word = "Sample")
}
result.await()
```

## 요약

* 애플리케이션에는 하나 이상의 프로세스가 있다. 각각은 적어도 하나의 스레드를 갖고 있고 코루틴은 스레드 안에서 실행된다.
* 코루틴은 재개될 때마다 다른 스레드에서 실행될 수 있지만 특정 스레드에만 국한될 수도 있다.
* 애플리케이션이 하나 이상의 스레드에 중첩돼 실행되는 경우는 동시적 실행이다.
* 올바른 동시성 코드를 작성하려면 서로 다른 스레드 간의 통신과 동기화 방법을 배워야 하며, 코틀린에서는 코루틴의 통신과 동기화 방법의 학습을 의미한다.
* 병렬 처리는 동시 처리 애플리케이션이 실행되는 동안 적어도 두 개 이상의 스레드가 같이 실행될 때 발생한다.
* 동시 처리는 병렬 처리 없이 일어날 수 있다. 현대적 처리 장치는 스레드 간에서 교차 배치 할 것이고 효과적으로 스레드를 중첩시킬 것이다.
* 동시성 코드를 작성하는 데에는 어려움이 많다. 대부분 올바른 통신과 스레드 동기화와 관련이 있는데 레이스 컨디션, 원자성 위반, 교착 상태 및 라이브락이 가장 일반적인 문제점이다.
* 코틀린은 동시성에 대해 현대적이고 신선한 접근 방식을 취했다. 코틀린을 사용하면 넌 블로킹이며, 가독성 있게 활용될 뿐만 아니라 유연한 동시성 코드를 작성할 수 있다.
