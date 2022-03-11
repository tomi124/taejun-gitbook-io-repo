# 자바 Multi Thread 환경에서 동시성 제어

## ##스레드란 <a href="#undefined" id="undefined"></a>

* 프로세스
  * 특정 작업을 수행하는 소프트웨어를 프로그램이라고 한다.
  * 프로그램이 실제로 실행되어, 메모리나 CPU와 같은 자원을 할당받으면 이를 프로세스라고 부름
  * 스레드는 이 프로세스를 구성하는 하나의 단위이다.\
    하나의 프로세스에는 여러 스레드가 작동하고 있을 수 있다.
  * 스레드
    * 스레드는 작업의 한 단위임
    * 프로세스는 독자적인 메모리를 할당받아서 서로 다른 프로세스끼리는 일반적으로 서로의 메모리 영역을 침범하지 못한다.
    * 하지만 프로세스 내부에 있는 여러 스레드들은 서로 같은 프로세스 내부에 존재하고 있기 때문에 같은 자원을 공유하여 사용할 수 있다.
    * 같은 자원을 공유할 수 있기 때문에 동시에 여러 가지 일을 같은 자원을 두고 수행할 수 있고, 이는 곧 병렬성의 향상으로 이어진다.
    * 여러 스레드가 동시에 하나의 자원을 공유하고 있기 때문에 이 자원을 두고 여러 가지 문제점이 발생할 수 있다.
      * 동시성 문제, 데드락과 같은 여러 가지 문제점을 해결해야 멀티스레드 환경에서 문제없는 프로그램을 제작할 수 있다.

## ##스레드 안정성이 깨지는 상황 <a href="#undefined" id="undefined"></a>

1. 조회수 계산 프로그램

*   100명의 스레드로 각각 100번 조회했을 때

    ```jsx
    public class CountingTest {
    	public static void main(String[] args) {
    		Count count = new Count();
    		for (int i = 0; i < 100; i++) {
    			new Thread(){
    				public void run(){
    					for (int j = 0; j < 100; j++) {
    						System.out.println(count.view());
    					}
    				}
    			}.start();
    		}
    	}
    }
    class Count {
    	private int count;
    	public int view() {return count++;}
    	public int getCount() {return count;}
    }
    ```
* 결과 : 10000번이 아닌, 더 적은 조회수가 나옴
* 이유 : 조회수를 증가시키는 로직이 두 번의 동작으로 이루어지는데 동시에 여러 스레드가 접근하여 첫 번째 동작할 때의 자원과 두 번째 동작할 때의 자원\
  상태가 변하기 때문

2.Count\
클래스의 view 메서드는 count++이라는 동작을 진행한다.\
count 변수의 값을 조회한다.\
조회한 count변수 값에 1을 더한 값을 저장한다.

![](https://media.vlpt.us/images/been/post/ba9f1849-b092-4ed4-aac1-3b0c63d6e4b0/image.png)

* 결과
  * 이러한 동작들 사이사이에 여러 스레드가 접근하게 되면 동시성 이슈가 발생할 수 있다..
  * 여러 스레드에서 동시에 count변수에 접근한다면 동시에 1번 동작을 진행하여 같은 count값을 조회할 것이고,\
    두 개의 스레드가 1을 더하는 조회 로직을 실행한다고 해도 2가 더해지는 것이 아닌 1만 더해지는 동작이 발생할 수 있다

## ##동시성을 제어하는 방법 : Lock <a href="#lock" id="lock"></a>

* 여러 쓰레드가 write하는 상황일때

![](https://media.vlpt.us/images/been/post/cead2e8b-c6d6-4be8-8d57-733f3280cb01/image.png)

1. 암시적 Lock : synchronized

: Lock을 건다.

* Lock을 적용하게 되면 하나의 스레드가 해당 메서드를 실행하고 있을 때 다른 메서드가 해당 메서드를 실행하지 못하고 대기하게 된다.
* 즉 한 번에 하나의 스레드만 접근할 수 있게 된다. → 여러 스레드가 동시에 접근할 수 없게 만들어 동시성 이슈를 막을 수 있음
* 단점 : 한 번에 하나의 스레드만 메서드를 실행시킬 수 있으므로 병렬성은 매우 낮아짐
* 문제가 된 view 메서드에 synchronized 키워드를 붙이면 암시적 락이 걸린다.

lock은 메서드, 변수에 각각 걸 수 있다.

* 메서드에 lock을 걸 경우 해당 메서드에 진입하는 스레드는 단 하나만 가능
* 변수에 lock을 걸 경우 해당 변수를 단 하나의 스레드만 참조할 수 있다.
* 단, 변수에 lock을 걸기 위해선 해당 변수는 객체여야 한다. int, long과 같은 기본형 타입에는 lock을 걸 수 없습니다.

1\) 메서드 Lock

```jsx
class Count {
 private int count;
 public synchronized int view() {return count++;}
}
```

2\) 변수 Lock

```jsx
class Count {
	 private Integer count = 0;
	 public int view() {
		 synchronized (this.count) {
			 return count++;
		 }
	 }
}
```

2.명시적 Lock : ReentrantLock\
: synchronized 키워드 없이 명시적으로 ReentrantLock을 사용하는 Lock이다.

* 언제사용? 해당 Lock의 범위를 메서드 내부에서 한정하기 어렵거나, 동시에 여러 Lock을 사용하고 싶을 때 사용
* 직접적으로 Lock 객체를 생성하여 사용할 수 있다.
* lock() : 다른 스레드가 해당 lock() 메서드 시작점에 접근하지 못하고 대기
* unlock() : 다른 메서드가 lock을 획득할 수 있게 된다.

명시적 Lock을 사용한 예제

```jsx
public class CountingTest {
 	public static void main(String[] args) {
 		Count count = new Count();
 		for (int i = 0; i < 100; i++) {
 			new Thread(){
 				public void run(){
 					for (int j = 0; j < 1000; j++) {
 						count.getLock().lock();
 						System.out.println(count.view());
 						count.getLock().unlock();
 					}
 				}
 			}.start();
 		}
 	}
}
class Count {
 	private int count = 0;
 	private Lock lock = new ReentrantLock();
 	public int view() {
 		return count++;
 	}
 	public Lock getLock(){
 		return lock;
 	};
}
```

#### #언제 synchoronized 식별자 사용? <a href="#synchoronized" id="synchoronized"></a>

1.하나의 객체에 여러개의 스레드가 접근해서 처리하고자 할때\
2.static으로 선언한 객체에 여러 스레드가 동시에 사용할때

* 변수를 공유할땐 static 변수사용, 하지만 static변수는 객체 생성할 수 없음
* Atomic은 기존변수에 동시성 제어를 하는것이 아니라 AtomicType객체를 생성해서 해당 AtomicType객체에 대한 동시성을 제어하는 것이다.

> 그러므로 새롭게 생성할 수 없는 Static변수나 기존 변수에 동시성을 제어하기 위해선 synchoronized 키워드를 붙여서 사용한다.

## ##동시성을 제어하는 방법 : Volatile <a href="#volatile" id="volatile"></a>

* volatile은 CPU 캐시 사용 막음 → 메모리에 접근해서 실제 값을 읽어오도록 설정 → 데이터 불일치를 막는다.
* 자원의 가시성을 책임진다
  * 메인 메모리에 저장된 실제 자원의 값을 볼 수 있는 것

\*\*자원의 가시성:

* 변수에 저장한 데이터를 읽었는데 이 데이터가 실제 데이터와 차이가 있을 수 있다.\
  메인 메모리에 저장된 실제 자원의 값을 볼 수 있는 개념을 자원의 가시성이라한다.

#### #사용상황 <a href="#undefined" id="undefined"></a>

* 멀티쓰레드 환경에서 하나의 쓰레드만 read\&write하고 나머지 쓰레드가 read하는 상황에 사용
  * Read : CPU cache에 저장된 값X , Main Memory에서 읽는다.
  * Write : Main Memory에 까지 작성
* 가장 최신의 값 보장

#### #Java volatile가 필요한 이유? <a href="#java-volatile" id="java-volatile"></a>

![](https://media.vlpt.us/images/been/post/b1b35d1f-d114-4c22-be1b-443cd5f36434/image.png)

* 멀티쓰레드 어플리케이션에서 volatile변수 사용 안할때 → Main Memory에서 읽은 변수 값을 CPU Cache에 저장
* 멀티쓰레드 환경에서 쓰레드가 변수값을 읽어올 때 각각의 CPU Cache에 저장된 값이 다르기 때문에 변수 값 불일치 문제가 발생

#### #Example <a href="#example" id="example"></a>

SharedObject를 공유하는 두 개의 Thread가 있다

```jsx
public class SharedObject {
 public int counter = 0;
}
```

* Thread-1는 counter 값을 더하고, 읽는 연산 (Read & Write)
* Thread-2는 counter 값을 읽기만(Only Read)

Thread-1은 counter값을 증가시킨다. 하지만 CPU Cache에만 반영, 실제로 Main Memory에는 반영X → Thread-2는 count값으로 0을 가져오는 문제 발생

![](https://media.vlpt.us/images/been/post/5abcb48c-3f27-4c82-8099-baf374011cd5/image.png)

#### #해결방법 <a href="#undefined" id="undefined"></a>

* volatile 키워드를 추가 → Main Memory에 저장하고 읽어옴 → 변수 값 불일치 문제해결

```jsx
public class SharedObject {
	public volatile int counter = 0;
}
```

#### #volatile 성능 <a href="#volatile" id="volatile"></a>

* volatile는 변수의 read와 write를 Main Memory에서 진행
* CPU Cache보다 Main Memory가 비용이 더 큼 (→ 변수 값 일치를 보장해야 하는 경우 사용 권장)

#### #정리 <a href="#undefined" id="undefined"></a>

* volatile
  * Main Memory에 read & write를 보장한다.
* 상황
  * 하나의 쓰레드만 read\&write하고 나머지 쓰레드가 read하는 상황에 적합
  * 변수의 값이 최신의 값으로 읽어와야 하는 경우.
* 성능
  * CPU Cache보다 Main Memory가 비용이 더 큼

## ##동시성을 제어하는 방법 : Atomic <a href="#atomic" id="atomic"></a>

* java.util.concurrent.atomic
* Atomic 클래스는 \*CAS(compare-and-swap)를 이용하여 동시성을 보장
* AtomicInteger는 synchronized 보다 적은 비용으로 동시성을 보장

#### #AtomicInteger 메서드 <a href="#atomicinteger" id="atomicinteger"></a>

* addAndGet (int delta) : 주어진 값을 현재 값에 원자 적으로 더합니다.
* compareAndSet (int expect, int update) : 현재 값이 예상 값과 같으면 원자 적으로 값을 지정된 업데이트 된 값으로 설정합니다.
* get () : 현재 값을 가져옵니다.\
  getAndAdd (int delta) : 주어진 값을 현재 값에 원자 적으로 더합니다.
* getAndIncrement () : 현재 값을 원자 적으로 1 씩 증가시킵니다. (현재값 반환 후 1씩 증가)
* incrementAndGet () : 현재 값을 원자 적으로 1 씩 증가시킵니다. (1씩 증가 후 현재값 반환)
* ...

#### #example <a href="#example-1" id="example-1"></a>

람다식

```jsx
public class AtomicIntExample {
		public static void main(String[] args) {
		ExecutorService executor = Executors.newFixedThreadPool(2);
		AtomicInteger atomicInt = new AtomicInteger();
		for(int i = 0; i < 10; i++){
			executor.submit(()->System.out.println("Counter- " + atomicInt.incrementAndGet())); // 1
		}
		executor.shutdown();
	}
}
```

```jsx
public class AtomicIntExample {
	public static void main(String[] args) {
		ExecutorService executor = Executors.newFixedThreadPool(2);
		AtomicInteger atomicInt = new AtomicInteger();
		CounterRunnable runnableTask = new CounterRunnable(atomicInt);
		for(int i = 0; i < 10; i++){
			executor.submit(runnableTask);
		}
		executor.shutdown();
	}
}
class CounterRunnable implements Runnable{
	AtomicInteger atomicInt;
	CounterRunnable(AtomicInteger atomicInt){
		this.atomicInt = atomicInt;
	}
	@Override
	public void run() {
		System.out.println("Counter- " + atomicInt.incrementAndGet());
	}
}
```

결과 :

```jsx
Counter- 1
Counter- 2
Counter- 3
Counter- 4
Counter- 5
Counter- 6
Counter- 7
Counter- 8
Counter- 9
Counter- 10
```

#### #CAS(Compare And Swap)알고리즘 <a href="#cascompare-and-swap" id="cascompare-and-swap"></a>

![](https://media.vlpt.us/images/been/post/547e45d9-1e41-424e-9993-0e6fd1fa893b/image.png)

AtomicInterger를 Decompile 하여 요약한 내용

```jsx
public class AtomicInteger extends Number implements java.io.Serializable {
	private volatile int value;
	
	public final int incrementAndGet() {
		int current;
		int next;
		do { 
			current = get(); 
			next = current + 1; 
		} while (!compareAndSet(current, next));
		return next;
	} 
	
	public final boolean compareAndSet(int expect, int update) { 
		return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
	} 
}
```

* incrementAndGet() 메소드 내부에서 **CAS알고리즘 로직구현**
* compareAndSet()의 결과 리턴 받아서 성공할때까지 while문 무한 루프를 돈다.
* compareAndSet() 내부에서는 compareAndSwapInt()를 호출하여 메모리에 저장되어진 값과 현재 CPU캐시에 저장된 expect 값을 비교하여 동일한 경우만 변경하려는 값 update로 쓰기를 수행한다.
* 다른 AtomicInteger 메소드 들도 거의 유사한 패턴으로 구현되어 있다.
* volatile int value; 선언부 인데 voliatile는 변수의 가시성 문제를 해결하기 위해서 사용된다.
* volatile 키워드가 붙어 있는 객체는 CPU캐시가 아닌 메인 메모리에서 값을 참조해온다.

### ##synchronized, volatile, AtomicInteger 비교 <a href="#synchronized-volatile-atomicinteger" id="synchronized-volatile-atomicinteger"></a>

synchronized

* 여러 쓰레드가 write하는 상황에 적합
* 가시성 문제해결 : synchronized블락 진입전 후에 메인 메모리와 CPU 캐시 메모리의 값을 동기화 하기 때문에 문제가 없도록 처리

volatile

* 하나의 쓰레드만 read\&write하고 나머지 쓰레드가 read하는 상황에 적합
* 가시성 문제해결 : CPU 캐시 사용 막음 → 메모리에 접근해서 실제 값을 읽어오도록 설정
  * Read : CPU cache에 저장된 값X , Main Memory에서 읽는다.
  * Write : Main Memory에 까지 작성

AtomicIntger

* 여러 쓰레드가 read\&write를 병행
* 가시성 문제해결 : CAS알고리즘
  * **현재 쓰레드에 저장된 값과 메인 메모리에 저장된 값을 비교 → 일치시 새로운 값으로 교체, 불일치시 실패하고 재시도**

## ##스레드 안전한 객체 사용 <a href="#undefined" id="undefined"></a>

#### #Concurrent 패키지 <a href="#concurrent" id="concurrent"></a>

* Executor
* ExecutorService
* Future
* Callable
* BlockingQueue
* ...

***

참조 :\
[https://dzone.com/articles/java-patterns-for-concurrency](https://dzone.com/articles/java-patterns-for-concurrency)\
[https://deveric.tistory.com/104](https://deveric.tistory.com/104)\
[https://javaplant.tistory.com/23](https://javaplant.tistory.com/23)
