# 이벤트 드리븐 아키텍쳐 & 쓰레드



### 이벤트 드리븐 <a href="#undefined" id="undefined"></a>

Event : 시스템 내, 외부에서 발생한 주목할 만한 상태의 변화\
\[a significant change in state]\
Event Driven : 이벤트 주도, 이벤트 기반\
architecture : 동작 방식

EDA : 이벤트를 기반으로 하는 아키텍처 설계 방식\
EDP : 이벤트를 기반으로 하는 프로그래밍

이벤트는 '명령'이 아닌 '관찰'로 생각하자.

#### 이벤트 드리븐 아키텍쳐 <a href="#undefined" id="undefined"></a>

EDA (Event-driven architecture) :\
이벤트의 생산, 감지, 소비 및 반응 또는 시스템 상태의 중대한 변화를 지원하는 소프트웨어의 모델 혹은 아키텍처 패러다임

복잡성과 역동성에 가장 효율적으로 대응할 수 있는 모델이다. (시시각각 발생하는 이벤트의 생성과 감지, 반응을 중심으로 구축되는 모델이기 때문이다.)

잘 체감이 안되지만, 프로그램이 어떤 유저액션(이벤트)에 대한 반응으로 동작하는 패턴이라고 생각된다.

가장 흔하게 예를 들을 수 있는것은 키보드, 마우스로 하는 동작이 이벤트이며, 이러한 이벤트는 또 다른 결과를 가져오고, 이러한 패턴

이벤트 기반 시스템은 이벤트 루프에서 이벤트를 처리하며, 이벤트 루프는 장치로부터 입력이나, 내부경보를 기다리고 있다가 활동이 발생하면, 이벤트를 생성시킨다. 발생한 이벤트의 데이터를 수집하고, 데이터를 필요한 이벤트 핸들러로 발송한다.

예시) 예매 사이트\
축구 경기를 예매 (동시간대)\
고객 A : A-1 좌석을 결제 요청 -> (해당 좌석이 실제 예매 가능인지 파악) -> 예매 완료,\
고객 B : A-1 좌석을 결제 요청 -> (해당 좌석이 실제 예매 가능인지 파악) -> 예매 진행중 -> 결제 불가 처리 (트렌젝션, rollback)

이렇게 다수의 사용자가 이벤트를 발생시키고, 이러한 결과는 또 다른 사용자 혹은 시스템을 실시간으로 변경을 발생시킨다.

데이터 무결성 (data integrity): 데이터는 완전한 수명 주기를 거치며 데이터의 정확성과 일관성을 유지하고 보증하는 것을 말한다.

구현\
![](https://media.vlpt.us/images/limprove89/post/4e43202f-62c4-458f-b04a-633ccdc144f1/Event-Driven-Architecture.png)

일반적으로 EDA는 3가지 필수 요소로 구성된다.

* Event Emitters : 시스템 내에서 발생하는 이벤트를 수집, 이벤트를 받아 채널로 전달
* Event channels : 특정 소비자에게 이벤트를 전달, 일정 수준의 전처리 수행 후 소비자에게 전달
* Event Consumers

이벤트 채널은 두 가지 토폴리지가 존재한다.

* 중재자 토폴리지
* 브로커 토폴리지

중재자 토폴리지는 단일 큐와 이벤트 중개자를 사용하여 이벤트를 처리, 일반적으로 이벤트 처리에 여러 단계가 필요할 경우 사용\
![](https://media.vlpt.us/images/limprove89/post/c2a7c82c-c4f0-4bee-8e28-8210c11e545b/Mediator-Topology.png)

브로커 토폴리지는 이벤트 플로우가 단순한, 중앙 집중식 토폴리지, 중재자와는 달리 이벤트 큐가 없다.

![](https://media.vlpt.us/images/limprove89/post/5e1fa4a6-aaab-43c6-a570-6ced8022f5d4/Broker-Topology.png)

이벤트 스트림을 생성하는 이벤트 생산자와 이벤트를 수신 대기하는 이벤트 소비자로 구성되며, 중간에 핸들링하는 채널이 존재

확장성이 뛰어난 애플리케이션을 생성하는데 사용되는 분산 비동기 아키텍쳐 패턴\
분산된 시스템 간에 이벤트를 생성, 발행 (publishing)하고 발행된 이벤트를 필요로하는 수신자에게 전송된다.\
이벤트를 수신한 수신자가 이벤트를 처리하는 형태의 시스템 아키텍쳐

#### 이벤트 드리븐 프로그래밍 <a href="#undefined" id="undefined"></a>

![](https://media.vlpt.us/images/limprove89/post/b65696a8-69bd-49b0-9001-eec5624f12eb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-08-02%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%202.02.35.png)

이벤트 기반 프로그래밍 (EDP: Event-driven programming)

과거에는 순차적, 절차적 프로그래밍이 일반적이었다.\
모든 프로그램의 흐름을 시간의 흐름대로 순서대로 정의하고 해석하였다. 현재도 비슷한 패턴은 동일

콘솔 프로그래밍 (Single-Tasking) 에서 사용자의 입력을 받는 타이밍이 있고, 해당 입력시 인터럽트가 발생하여도 경우의 수가 적었다.\
이 말을 정리하면, 흐름을 예측할 수 있었고, 예측 불가한 예외는 소수였다고 볼 수 있다. 따라서 순차적 프로그래밍이 적합하였다.

하지만 현대적 프로그램 환경(윈도우 프로그래밍 Multi-Tasking : GUI 환경)에서는 이 패턴은 잘 맞지 않았다. 현대의 환경에서는 흐름을 예측하기가 매우 어렵다. (이벤트가 몹시 다양하게 발생 : 마우스, 키보드 타이밍과 값 등 예외가 매무 많다.)\
이러한 상황에서도 순차적 프로그래밍은 가능하다. 모든 이벤트를 예외로 처리하는 방식으로 적용 가능하다. 실제로 Win32 API가 이런식으로 구현되었다.

![](https://media.vlpt.us/images/limprove89/post/42d0a271-3c45-4c21-900a-252b8442b551/011760345127C02627.png)

![](https://media.vlpt.us/images/limprove89/post/232c469d-377d-4f8d-a571-b437a359a21b/2025353B5127C1941C.png)

싱글 테스킹서는 한번에 하나의 프로그램만을 실행하고, 멀티 테스킹에서는 동시에 여러 프로그램이 실행된다. 콘솔은 자기 자신만을 고려하여 프로그램을 작성하면 되지만, 윈도우는 동시에 수행되는 다른 프로그램을 고려하여 개발을 진행해야 한다. 이런 부분을 운영체제가 알아서 처리해주며, 그러한 규칙을 정한것이 EDP이다.

이벤트 -> 이벤트 처리\
개발자는 메세지를 처리하는 함수를 작성하고, 윈도우를 생성하여 출력만 진행하면 된다.

![](https://media.vlpt.us/images/limprove89/post/576b3021-7d01-4875-92cf-b5ca36b8a865/01-13-638.jpg)

컴퓨터 프로그램 중 특정 이벤트에 반응하여 동작을 변경하는 방식을 Event-driven 방식이라고 한다.

프로그램은 이벤트를 처리하기 위한 이벤트 핸들러를 가지며, 이벤트 루프라는 곳에서 이를 처리한다.

#### 활용 예시 <a href="#undefined" id="undefined"></a>

배달의 민족 - 배민찬 도입\
[https://youtu.be/Dn5irt7bClM?t=1349](https://youtu.be/Dn5irt7bClM?t=1349)

이벤트 생산과 소비를 통한 느슨한 결합?

이벤트는 이벤트 생산자와 이벤트 소비자, 둘을 연결해주는 채널 이 세가지 역활이 존재한다.\
생산자는 이벤트를 발행하고, 소비자는 누가 이벤트를 발행했는지 알 필요 없이, 이벤트를 소비한다.\
중간의 채널은 두 이벤트가 적절히 연결될 수 있도록 해준다.\
이렇게 생산자와 소비자의 결합이 분리되는 형태를 느슨한 결합이라고 하며, 언어 혹은 프레임워크에서 이러한 형태를 구현하기 쉽도록 이벤트 매커니즘을 제공한다.\
\[Java Spring 예시]

```
// Event Producer 이벤트 생산, 스프링에 application context object의 대다수 구현
interface ApplicationEventPublisher {

	void publishEvent(ApplicationEvent event);
    void publishEvent(Object event);
}

// Event consumer 이벤트 소비, 이벤트를 받아 처리하는 역활
interface ApplicationListener<E entends ApplicationEvent>
extends EventListener {
	void onApplicationEvent(E event);
}

class ApplicationComponent {
	@EventListener
    void handle(orderCompleted orderCompleted) {
}

// Event Channel
sping 내부 구현
```

!\[javascript eventloop][http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDMwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDMwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

![](https://media.vlpt.us/images/limprove89/post/4a00feae-2918-4e81-8134-90d18cbc5a07/11.png)

Event-driven 의 구조는 3가지로 분류된다.

* 이벤트를 발생시키는 객체 (EventEmitter)
* 이벤트를 관리하는 객체 (EventDispatcher)
* 이벤트가 발생했을 때 샐행되는 객체 (EventHandler)

eventEmitter가 이벤트를 발생시키면, eventEmitter에서 실행되는 매소드들이 EventHandler가 되며, 이 둘을 관리하는 eventDispatcher가 존재,\
[http://wiki.sys4u.co.kr/display/SOWIKI/1.Event-driven+programming](http://wiki.sys4u.co.kr/display/SOWIKI/1.Event-driven+programming)

### 쓰레드 <a href="#undefined" id="undefined"></a>

|      | 동시성                  | 병렬성                |
| ---- | -------------------- | ------------------ |
| 용어차이 | 논리적                  | 물리적                |
| 뜻    | 동시에 실행되는 것처럼 보이는 것   | 실제로 동시에 실행 처리 되는 것 |
| 코어   | 싱글 코어, 멀티 코어에서 사용 가능 | 멀티 코어에서만 사용 가능     |

#### 프로세스와 쓰레드의 차이 <a href="#undefined" id="undefined"></a>

![](https://media.vlpt.us/images/limprove89/post/e5b6cea7-9052-4e1d-ad95-473f2bb7cac7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-08-04%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%201.32.05.png)

프로그램(Program) 이란?\
“어떤 작업을 위해 실행할 수 있는 파일”

프로세스(Process) 란?\
“컴퓨터에서 연속적으로 실행되고 있는 컴퓨터 프로그램”

* 메모리에 올라와 실행되고 있는 프로그램의 인스턴스(독립적인 개체)
* 운영체제로부터 시스템 자원을 할당받는 작업의 단위
* 즉, 동적인 개념으로는 실행된 프로그램을 의미한다.

할당받는 시스템 자원의 예

* CPU 시간
* 운영되기 위해 필요한 주소 공간
* Code, Data, Stack, Heap의 구조로 되어 있는 독립된 메모리 영역

특징

* 프로세스는 각각 독립된 메모리 영역(Code, Data, Stack, Heap의 구조)을 할당받는다.
* 기본적으로 프로세스당 최소 1개의 스레드(메인 스레드)를 가지고 있다.
* 각 프로세스는 별도의 주소 공간에서 실행되며, 한 프로세스는 다른 프로세스의 변수나 자료구조에 접근할 수 없다.
* 한 프로세스가 다른 프로세스의 자원에 접근하려면 프로세스 간의 통신(IPC, inter-process communication)을 사용해야 한다.\
  Ex. 파이프, 파일, 소켓 등을 이용한 통신 방법 이용

스레드(Thread) 란 ?\
“프로세스 내에서 실행되는 여러 흐름의 단위”

* 프로세스의 특정한 수행 경로
* 프로세스가 할당받은 자원을 이용하는 실행의 단위\
  ![](https://media.vlpt.us/images/limprove89/post/5b2a2a69-ba4f-4d41-9adf-a113119a6a68/thread.png)
* 스레드는 프로세스 내에서 각각 Stack만 따로 할당받고 Code, Data, Heap 영역은 공유한다.
* 스레드는 한 프로세스 내에서 동작되는 여러 실행의 흐름으로, 프로세스 내의 주소 공간이나 자원들(힙 공간 등)을 같은 프로세스 내에 스레드끼리 공유하면서 실행된다.
* 같은 프로세스 안에 있는 여러 스레드들은 같은 힙 공간을 공유한다. 반면에 프로세스는 다른 프로세스의 메모리에 직접 접근할 수 없다.
* 각각의 스레드는 별도의 레지스터와 스택을 갖고 있지만, 힙 메모리는 서로 읽고 쓸 수 있다.
* 한 스레드가 프로세스 자원을 변경하면, 다른 이웃 스레드(sibling thread)도 그 변경 결과를 즉시 볼 수 있다.

프로세스 내에 스레드가 포함되어 있다. 프로세스가 집이라면 스레드는 가족 구성원, 프로세스는 실행중인 프로그램 객체 자체를 말하고, 하나의 실행 흐름 자체를 스레드라고 한다.\
![](https://media.vlpt.us/images/limprove89/post/3c61e85a-c9a5-4f89-b256-5425805a2a43/237976395901A1A004.png)

#### 멀티 프로세스와 멀티 스레드의 차이 <a href="#undefined" id="undefined"></a>

**멀티 프로세스**

멀티 프로세싱이란

* 하나의 응용프로그램을 여러 개의 프로세스로 구성하여 각 프로세스가 하나의 작업(태스크)을 처리하도록 하는 것이다.

장점

* 여러 개의 자식 프로세스 중 하나에 문제가 발생하면 그 자식 프로세스만 죽는 것 이상으로 다른 영향이 확산되지 않는다.

단점

* Context Switching에서의 오버헤드\
  Context Switching 과정에서 캐쉬 메모리 초기화 등 무거운 작업이 진행되고 많은 시간이 소모되는 등의 오버헤드가 발생하게 된다.\
  프로세스는 각각의 독립된 메모리 영역을 할당받았기 때문에 프로세스 사이에서 공유하는 메모리가 없어, Context Switching가 발생하면 캐쉬에 있는 모든 데이터를 모두 리셋하고 다시 캐쉬 정보를 불러와야 한다.
* 프로세스 사이의 어렵고 복잡한 통신 기법(IPC)\
  프로세스는 각각의 독립된 메모리 영역을 할당받았기 때문에 하나의 프로그램에 속하는 프로세스들 사이의 변수를 공유할 수 없다.
* Context Switching ?\
  CPU에서 여러 프로세스를 돌아가면서 작업을 처리하는 데 이 과정을 Context Switching라 한다.\
  구체적으로, 동작 중인 프로세스가 대기를 하면서 해당 프로세스의 상태(Context)를 보관하고, 대기하고 있던 다음 순서의 프로세스가 동작하면서 이전에 보관했던 프로세스의 상태를 복구하는 작업을 말한다.

**멀티 스레드**

멀티 스레딩이란

* 하나의 응용프로그램을 여러 개의 스레드로 구성하고 각 스레드로 하여금 하나의 작업을 처리하도록 하는 것이다.
* 윈도우, 리눅스 등 많은 운영체제들이 멀티 프로세싱을 지원하고 있지만 멀티 스레딩을 기본으로 하고 있다.
* 웹 서버는 대표적인 멀티 스레드 응용 프로그램이다.

장점

* 시스템 자원 소모 감소 (자원의 효율성 증대)\
  프로세스를 생성하여 자원을 할당하는 시스템 콜이 줄어들어 자원을 효율적으로 관리할 수 있다.
* 시스템 처리량 증가 (처리 비용 감소)\
  스레드 간 데이터를 주고 받는 것이 간단해지고 시스템 자원 소모가 줄어들게 된다.\
  스레드 사이의 작업량이 작아 Context Switching이 빠르다.
* 간단한 통신 방법으로 인한 프로그램 응답 시간 단축\
  스레드는 프로세스 내의 Stack 영역을 제외한 모든 메모리를 공유하기 때문에 통신의 부담이 적다.

단점

* 주의 깊은 설계가 필요하다.
* 디버깅이 까다롭다.
* 단일 프로세스 시스템의 경우 효과를 기대하기 어렵다.
* 다른 프로세스에서 스레드를 제어할 수 없다. (즉, 프로세스 밖에서 스레드 각각을 제어할 수 없다.)
* 멀티 스레드의 경우 자원 공유의 문제가 발생한다. (동기화 문제)
* 하나의 스레드에 문제가 발생하면 전체 프로세스가 영향을 받는다.

#### 멀티 프로세스 대신 멀티 스레드를 사용하는 이유? <a href="#undefined" id="undefined"></a>

* 멀티 프로세스 대신 멀티 스레드를 사용하는 것의 의미?\
  쉽게 설명하면, 프로그램을 여러 개 키는 것보다 하나의 프로그램 안에서 여러 작업을 해결하는 것이다.
* 여러 프로세스(멀티 프로세스)로 할 수 있는 작업들을 하나의 프로세스에서 여러 스레드로 나눠가면서 하는 이유?
* 자원의 효율성 증대\
  멀티 프로세스로 실행되는 작업을 멀티 스레드로 실행할 경우, 프로세스를 생성하여 자원을 할당하는 시스템 콜이 줄어들어 자원을 효율적으로 관리할 수 있다.\
  –> 프로세스 간의 Context Switching시 단순히 CPU 레지스터 교체 뿐만 아니라 RAM과 CPU 사이의 캐쉬 메모리에 대한 데이터까지 초기화되므로 오버헤드가 크기 때문\
  스레드는 프로세스 내의 메모리를 공유하기 때문에 독립적인 프로세스와 달리 스레드 간 데이터를 주고 받는 것이 간단해지고 시스템 자원 소모가 줄어들게 된다.
* 처리 비용 감소 및 응답 시간 단축\
  또한 프로세스 간의 통신(IPC)보다 스레드 간의 통신의 비용이 적으므로 작업들 간의 통신의 부담이 줄어든다.\
  –> 스레드는 Stack 영역을 제외한 모든 메모리를 공유하기 때문\
  프로세스 간의 전환 속도보다 스레드 간의 전환 속도가 빠르다.\
  –> Context Switching시 스레드는 Stack 영역만 처리하기 때문

**멀티쓰레드 프로그래밍이 어려운 이유?**

자원의 공유, 문제가 발생할 확률이 매우 높음, 프로그램 복잡성, 동기화 문제

시분할 (time division multiflxing)\
CPU는 하나이지만, 쓰레드의 수는 8개지만, 쓰레드는 계속적으로 생성이 가능하다. 이는 시분할 시스템이라는 개념이 있는데, 운영체제는 여러 개의 프로그램을 동시에 실행하기 위하여 CPU에 여러 프로그램 코드를 할당하고, 각 프로그램의 코드를 잘게 나누어서 반복적으로 실행한다. 이 때, 작업의 중단했던 시점의 기록을 '컨텍스트', 작업을 전환하는 것을 '컨텍스트 스위칭'이라고 한다.

선점형, 협력형\
선점형 멀티테스킹 -> CPU 자원을 커널이 관리\
협력형 멀티테스킹 -> 스스로 컨텍스트 전환 시점 판단

스레드와 관련된 문제들 (Threading Issues)

fork 문제 : 어떤 프로세스 내의 스레드가 fork를 호출하면 모든 스레드를 가진 프로세스를 생성할 것인지, 아니면 fork를 요청한 스레드만 가진 프로세스를 생성할 것인지 하는 문제이다. 유닉스에서는 각각 2가지 버전의 fork를 지원하고 있다.

exec 문제 : fork를 통해 모든 스레드를 복제하고 난 후, exec를 수행한다면 모든 스레드들이 초기화된다. 그렇다면 교체될 스레드를 복제하는 작업은 필요가 없기 때문에 애초에 fork를 요청한 스레드만을 복제했어야 한다. 한편, fork를 한 후에 exec를 수행하지 않는다면 모든 스레드를 복제할 필요가 있는 경우도 있다.

스레드 풀 (다중 스레드 서버의 문제)\
서비스할 때마다 스레드를 생성하는데 소요되는 시간\
동시에 실행할 수 있는 최대 스레드 수가 몇 개까지 가능할 수 있는 것인지 -> 이런 문제들을 해결해 주는 것이 스레드 풀(pool)

* 프로세스를 시작할 때 아예 일정한 수의 스레드들을 미리 풀로 만들어두는 것
* 평소에는 하는 일 없이 일감을 기다리다 요청이 들어오면 풀의 한 스레드에게 서비스 요청을 할당, 요청을 다 서비스해 주었으면 그 스레드는 다시 풀로 들어가 다음 작업을 기다림

![](https://media.vlpt.us/images/limprove89/post/ccfa3e53-bd68-4a25-bf5a-7dc88d3ee42f/img1.daumcdn-2.png)

#### 멀티 쓰레드 프로그래밍 <a href="#undefined" id="undefined"></a>

```
public class RamenProgram {
    
	public static void main(String args[]) {
        try {
            RamenCook ramenCook = new RamenCook(Integer.parseInt(args[0]));
    	new Thread(ramenCook, "A").start();
        new Thread(ramenCook, "B").start();
        new Thread(ramenCook, "C").start();
        new Thread(ramenCook, "D").start();
        } catch (Exception e) {
        e.printStackTrace();
    	}
    }
}

class RamenCook implements Runnable {
	private int ramenCount;
    private String[] burners = {"_","_","_","_"};
    
    public RamenCook(int count) {
    ramenCount = count;
    }
    
    @Override
    public void run() {
    	while (ramenCount > 0) {
        	synchronized(this) {
            	ramenCount--;
                System.out.println(
                Thread.currentThread().getName()
                + ": " + ramenCount + "개 남음");
            }
            
            for (int i = 0; i < burners.length; i++) {
            	if (!burners[i].equals("_")) continue;
                
                synchronized(this) {
                // if (burners[i].equals("_")) {
                	burners[i] = Thread.currentThread().getName();
                    System.out.println(
                    	"		"
                        + Thread.currentThread().getName()
                        	+ ": [" + (i + 1) + "]번 버너 ON");
                     showBurners();
                  // }
                 }
                
                 try {
                     Thread.sleep(2000);
                     } catch (Exception e) {
                     e.printStackTrace();
                     }
                     
                     synchronized(this) {
                         burners[i] = "_";
                         System.out.println (
                         "		"
                         	+ Thread.currentThread().getName()
                            	+ ": [" + (i + 1) + "]번 버너 OFF");
                          showBurners();
                      }
                      break;
                  }
                  
                  	try {
                    Thread.sleep(Math.round(1000 * Math.random()));
                    } catch (Exception e) {
                      e.printStackTrace();
                      }
                  }
              }
              
              private void showBurners() {
              	String stringToPrint
                = "		";
                for (int i = 0; i < burners.length; i++) {
                	stringToPrint += (" " + burners[i]);
                    }
                    System.out.println(stringToPrint);
                    }
                }
```



출처: [https://velog.io/@limprove89/%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EA%B8%B0%EB%B0%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EC%93%B0%EB%A0%88%EB%93%9C](https://velog.io/@limprove89/%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EA%B8%B0%EB%B0%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EC%93%B0%EB%A0%88%EB%93%9C)&#x20;
