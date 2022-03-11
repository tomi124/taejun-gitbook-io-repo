# Thread vs Coroutine



> **🤚🏻 가볍게 다시 짚어보기**
>
> **동기** : 어떤 요청을 보낸 뒤, 그 요청의 결과값을 얻기까지 작업을 멈추는 것\
> **비동기** : 어떤 요청을 보낸 뒤, 그 요청의 결과값이 오기까지 멈추지 않고 또 다른 일을 수행하는 것

**쓰레드와 코루틴의 차이점**에 관해 알아보기 전에, 몇 가지 선수지식을 살펴보자!

### Process with Thread <a href="#process-with-thread" id="process-with-thread"></a>

> **Process** : 보조기억장치의 '프로그램'이 **메모리 상으로 적재되어 실행되면 '프로세스'**가 된다.\
> **Thread** : 같은 Process 내에서 실행되는 **여러 작업 (흐름)의 단위**

![](https://media.vlpt.us/images/haero\_kim/post/2981356e-33f1-4d13-8611-9cd6252c3618/%E1%84%89%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%83%E1%85%B31.png)

**Process** 는 독립된 메모리 영역인 **'힙'을 할당**받는다.

**Thread** 는 **Process 하위에 종속되는 보다 작은 단위**이고, 각 **쓰레드는 독립된 메모리 영역인 스택(Stack)** 을 갖는다. Thread 를 하나 생성하면, 하나의 스택 메모리가 생기는 것이다. 각 쓰레드는 **다른 쓰레드에게 스택 메모리를 공유할 수 없다**. 하지만 프로세스의 **힙은 속한 모든 Thread 가 공유**할 수 있다.

### 동시성 vs 병렬성 <a href="#vs" id="vs"></a>

#### 동시성 (Concurrency) <a href="#concurrency" id="concurrency"></a>

동시성 프로그래밍은 말 그대로 **동시에 여러 작업**을 수행하는 것이다. 하지만 눈으로 보기에 동시에 실행되는 것이지, 사실 **시분할(Interleaving) 기법을 활용**하여 여러 작업을 **조금씩 나누어서 번갈아가며 실행**하는 것이다.

![](https://media.vlpt.us/images/haero\_kim/post/45350de0-d7ab-478b-99ca-c93df6362b0b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-19%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.24.19.png)

위 예제를 살펴보면, Task 1 과 Task 2 를 **잘개 쪼개어 번갈아가며 수행하여 사용자 입장**에선 두 작업이 **동시에 일어나는 것처럼 느껴지게** 된다. 따라서 **10분 + 10분, 총 20분이 소요**된다. (Context Switching 고려 X)

#### 병렬성 (Parallelism) <a href="#parallelism" id="parallelism"></a>

병렬성 프로그래밍은 시분할 기법없이 찐으로 **여러 작업을 한 번에 동시에 수행**하는 것인데, **자원(CPU 코어)의 입장에선 자기는 자기 할 일 1개**를 하는 것 뿐이다. **즉, 병렬성은 '자원(CPU 코어)이 여러 개' 일 때 가능**하다.

![](https://media.vlpt.us/images/haero\_kim/post/9b6fd6a4-ac11-40cc-8e3f-db2cfb75cf1f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-19%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.24.33.png)

위 예시를 보자. Task 1 과 Task 2 가 **병렬적으로 동시에 수행**된다. 이는 '멀티코어' 시스템에서 가능한 이야기이다. **코어 각각이 Task 1, Task 2 를 맡아 작업을 수행**하면, 완전히 동시에 수행되는 것이다. 따라서, 각각 태스크가 10분씩이라면, **최종적으로 10분이 소요**되는 것이다. 이 경우 Context Switching 이 일어나지 않는다.

#### 동시성 병렬성 이야기는 왜..? <a href="#undefined" id="undefined"></a>

Thread 와 Coroutine 은 모두 '**동시성 프로그래밍**'을 위한 기술이다. 이제부터, '동시성'을 보장하기 위해 등장한 두 기술인 Thread 와 Coroutine 의 **개념과 차이점**에 대하여 자세히 알아보자.

***

### Thread vs Coroutine <a href="#thread-vs-coroutine" id="thread-vs-coroutine"></a>

#### 개요 <a href="#undefined" id="undefined"></a>

* 쓰레드는 **각 태스크에 해당하는 스택 메모리를 할당**받는다. 그리고 **여러 작업을 동시에 수행**해야할 때 OS 는 **어떤 쓰레드 작업을 먼저 수행**할지, **어떤 쓰레드를 더 많이 수행**해야 **효율적**인지에 대한 **스케쥴링** (**선점 스케쥴링, Preempting Scheduling**) 을 해야 한다.
* **Coroutuine** 은 **Lightweight Thread** 라고 부른다. (코틀린 기준, 공식 문서에도 이렇게 설명되어 있다) 마찬가지로 **작업 하나하나를 효율적으로 분배해서 동시성을 보장하는 것을 목표**로 하지만, 작업 하나하나에 **Thread 를 할당하는 것이 아닌** '**Object**' 를 **할당**해주고, 이 **Object** 를 자유롭게 **스위칭함으로써 Context Switching 비용을 대폭 줄인** 것이다.

#### Thread <a href="#thread" id="thread"></a>

* 작업 하나하나의 단위 : **Thread**
  * 각 Thread 가 독립적인 **Stack 메모리 영역** 가짐
* 동시성 보장 수단 : **Context Switching**
  * 운영체제 커널에 의한 **Context Switching** 을 통해 동시성 보장
  * **블로킹** (Blocking) : Thread A 가 Thread B 의 **결과가 나오기까지 기다려야 한다**면, Thread A 은 **블로킹**되어 Thread B 의 **결과가 나올 때 까지 해당 자원을 못 씀**

아래 예시 사진을 살펴보자.

![](https://media.vlpt.us/images/haero\_kim/post/1d041a32-90b2-4c4d-b89b-b125494089b3/context-switch-between-threads.png)

**Thread A** 에서 **Task 1 을 수행**하다가 **Task 2 의 결과가 필요**할 때, 비동기적으로 **Thread B 를 호출**을 하게 된다. 이 때 **Thread A 는 블로킹**되고, **Thread B 로 Context Switching** 이 일어나 **Task 2** 를 수행한다. **Task 2 가 완료**되면, 다시 **Thread A 로 Context Switching** 이 일어나 결과값을 **Task 1** 에 반환한다.

동시에 같이 수행할 **Task 3, Task 4** 는 각각 **Thread C 와 D** 에 **할당**되고, 총체적으로 봤을 때 커널 입장에서 **Preempting Scheduling** 을 통하여 **각 태스크를 얼만큼 수행할지, 혹은 무엇을 먼저 수행할지를 결정**하여 알맞게 **동시성을 보장**하게 되는 것이다.

#### Coroutine <a href="#coroutine" id="coroutine"></a>

* 작업 하나하나의 단위 : **Coroutine Object**
  * 여러 작업 각각에 Object 를 할당함
  * Coroutine Object 도 엄연한 객체이기 때문에 **JVM Heap 에 적재** 됨 (코틀린 기준)
* 동시성 보장 수단 : **Programmer Switching (No-Context Switching)**
  * 프로그래머의 코드를 통해 **Switching** 시점을 **마음대로** 정함 (OS 관여 X)
  * **Suspend** (Non-Blocking) : Object 1 이 Object 2 의 **결과가 나오기까지 기다려야 한다**면, Object 1 은 **Suspend** 되지만, Object 1 을 수행하던 **Thread 는 그대로 유효하기** 때문에 Object 2 도 Object 1 과 **동일한 Thread 에서 실행**될 수 있음

이해를 위해 아래 예시 사진을 살펴보자.

![](https://media.vlpt.us/images/haero\_kim/post/96cd2cfd-4539-4417-9f13-ab905446e0e2/no-context-switch-between-coroutines.png)

작업 단위는 **Coroutine Object** 이므로, **Task 1 을 수행**하다가 **비동기 작업 Task 2 가 발생**하더라도, **Context Switching 없이** **같은 Thread 에서 Task 2 를 수행할 수 있고**, 맨 오른쪽 경우처럼 **하나의 Thread 에서 여러 Coroutine Object 들을 같이 수행**할 수도 있다. **한 쓰레드에서 다수의 Coroutine 을 수행**할 수 있고, **Context Switching 이 필요없는 특성**에 따라 **Coroutine 을 Light-weight Thread 라고 부르는 것**이다.

그런데 위 경우를 다시 보자. **Thread A 와 C 가 동시에 수행되는 모습**이다. 이러면 결국 **동시성 보장을 위해서 Context Switching 이 필요한 경우**다. 따라서, **Coroutine 의 'No-Context Switching' 장점**을 극강으로 끌어올리기 위해, **단일 Thread 에서 여러 Coroutine Object 를 컨트롤하는 것이 좋다**. (또는 권장한다)

> #### 💡 여기서 알 수 있는 점 <a href="#undefined" id="undefined"></a>
>
> **Coroutine 은 Thread 의 대안이 아니라, Thread 를 더 잘게 쪼개어 사용하기 위한 개념이다.**
>
> * 작업의 단위를 **Object** 로 축소하면서 하나의 **Thread** 가 다**수의 코루틴을 다룰 수 있기 때문에,** 작업 하나하나에 Thread 를 할당하며 **메모리 낭비, Context Switching 비용 낭비를 할 필요가 없음**
> * 같은 프로세스 내의 **Heap 에 대한 Locking 걱정 또한 사라짐**

한 마디로, 코루틴은 **프로그래머의 목적과 의도대로 효율성을 올릴 수 있는 동시성 보장** 기법이다.

### 마무리 <a href="#undefined" id="undefined"></a>

과거 비동기 처리라 함은 **DB 트랜잭션, 네트워킹 요청** 등을 이야기했는데, 최근에는 이러한 작업이 아니더라도 **비동기 처리를 접목하는 것을 쉽게 찾아볼** 수 있다. 그만큼 비동기 처리가 **간편해졌다**는 이야기인 것 같다. 그리고 이 비동기 처리가 간편해짐에 **'코루틴'이 많은 기여**를 한 것으로 보인다. 왜냐하면, 프로세스의 작은 작업 단위인 Thread 를 더 잘게 쪼개어 관리할 수 있기 때문이다.

핵심은 **코루틴이 쓰레드의 대안으로 등장한 것이 아니고**, 위와 같은 차이점이 있는 **또다른 동시성 보장 방법**으로 알아두면 된다는 점이다. Thread 기법에 비해 **다방면에서 비용이 절감**되기 때문에, 적극적으로 활용해보자.



출처: [https://velog.io/@haero\_kim/Thread-vs-Coroutine-%EB%B9%84%EA%B5%90%ED%95%B4%EB%B3%B4%EA%B8%B0](https://velog.io/@haero\_kim/Thread-vs-Coroutine-%EB%B9%84%EA%B5%90%ED%95%B4%EB%B3%B4%EA%B8%B0)&#x20;
