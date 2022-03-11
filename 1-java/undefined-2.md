# 자바 싱글톤 패턴



![post-thumbnail](https://media.vlpt.us/images/kyle/post/1c27d681-826c-47a0-a522-88faaacbe655/banner-image%20\(8\).png)

안녕하세요. 이번 포스팅에서는 자바의 싱글톤 패턴에 대해서 이야기하고자 합니다. 무한으로 존재하는 자원은 존재하지 않고 컴퓨터가 제공하는 자원 마찬가지로 제한되어 있습니다. 이러한 상황에서 인스턴스가 남용되는 것은 바람직하지 않고 하나의 자원으로 모두가 공유해서 사용해야하는 경우 싱글톤패턴은 유용한 방법이 될 수 있습니다.

### 정의 <a href="#undefined" id="undefined"></a>

> 소프트웨어 디자인 패턴에서 싱글턴 패턴(Singleton pattern)을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다. 이와 같은 디자인 유형을 싱글턴 패턴이라고 한다. 주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.

위키피디아의 정의와 같이 싱글턴 패턴은 하나의 객체만을 생성해 이후에 호출된 곳에서는 생성된 객체를 반환하여 프로그램 전반에서 하나의 인스턴스만을 사용하게 하는 패턴입니다.

### 사용 및 예제 <a href="#undefined" id="undefined"></a>

하나의 프린터기를 여러명의 사원이 공유하며 사용하는 경우 어떤식으로 코딩하면 좋을까요? 이 경우 싱글턴패턴이 유용하게 사용될 수 있는데요 아래의 예제를 보겠습니다.

```java
    package kail.study.java.design.single;
    
    public class Printer {
    	private static Printer printer = null;
    
    	private Printer(){}
    
    	public static Printer getInstance() {
    		if(printer == null) {
    			printer = new Printer();
    		}
    		return printer;
    	}
    
    	public void print(String input) {
    		System.out.println(input);
    	}
    }
```

위의 예제는 싱글턴 패턴이 적용된 예제입니다. 기본생성자를 `private` 를 사용하여 생성을 불가능하게 하고 getInstance를 통해서만 생성이 가능합니다. getInstance는 내부적으로 생성되지 않았다면 생성하고, 기존에 생성된 값이 존재한다면 생성된 인스턴스를 리턴하는 형태로 프로그램 전반에 걸쳐 하나의 인스턴스를 유지합니다.

또한 참고할 점은 인스턴스를 제공하는 메서드와 인스턴스 변수 모두 Static으로 선언된 정적 변수 및 메서드입니다. 당연히 기본생성자를 통해 생성할 수 없기 때문에 외부에서 인스턴스에 접근하려면 클래스 변수 및 메서드에 접근을 허용해야하기 때문에 두 메서드는 정적타입으로 선언되어 있습니다.

### 문제점 <a href="#undefined" id="undefined"></a>

하지만 위와 같은 코드에는 문제점이 있습니다. Multi-Thread 환경에서 안전하지 않기 때문인데요. 여러 쓰레드가 공유되고 있는 상황에서는 아래의 블럭에서 조건문이 동시에 두번 돌 수 있기때문에 하나의 인스턴스가 아닌 여러개의 인스턴스가 발생 할 위험이 있습니다.

```java
    	public static Printer getInstance() {
    		if(printer == null) {
    			printer = new Printer();
    		}
    		return printer;
    	}
    
```

뿐만 아니라 인스턴스가 상태를 유지해야하는 상황에서 싱글톤은 더 고려해야할 점이 많습니다. 아래의 예제에서 count값은 각기 다른 쓰레드에서 공유하고 있고 서로 다른 프로세스에서 처리하고 있기 때문에 값이 일관되지 않을 수 있습니다.

```java
    public class Printer {
    	private static Printer printer = null;
    	private int count = 0;
    
    	private Printer(){}
    
    	public static Printer getInstance() {
    		if(printer == null) {
    			printer = new Printer();
    		}
    		return printer;
    	}
    
    	public void print(String input) {
    		count++;
    		System.out.println(input + "count : "+ count);
    	}
    }
```

### 해결 <a href="#undefined" id="undefined"></a>

멀티쓰레드 환경에서 싱글톤의 문제를 해결 할 수 있는 방법은 두가지가 있습니다.

* 정적 변수에 인스턴스를 만들어 바로 초기화 하는 방법
* 인스턴스를 만드는 메서드에 동기화하는 방법

1 . **정적 변수에 인스턴스를 만들어 바로 초기화 하는 방법**

정적 변수는 객체가 생성되기 전 클래스가 메모리에 로딩할 때 만들어져 초기화가 한 번만 실행된다. 또한 정적 변수는 프로그램이 시작될 때부터 종료될 때까지 없어지지 않고 메모리에 계속 상주하며 클래스에서 생성된 모든 객체에서 참조할 수 있다. 따라서 기존에 조건문에서 체크하던 부분이 원천적으로 제거된다.

* 하지만 이 경우에도 Count 값은 각기 다르게 접근하기 때문에 쓰레드마다 값이 달라진다. 객체 생성 자체는 로드 시점에서 결정되어 하나의 객체만을 사용하지만 Count에 접근하는 것은 동시적으로 접근하기 때문에 그렇다. 이를 해결하기 위해서는 아래와 같이 `synchronized` 라는 키워드를 통해 여러 쓰레드에서 동시에 접근하는 것을 막는 방법으로 이를 해결 할 수 있다.

```java
    package kail.study.java.design.single;
    
    public class Printer {
    	private static Printer printer = new Printer();
    	private static int count = 0;
    
    	private Printer(){}
    
    	public static Printer getInstance() {
    		return printer;
    	}
    
    	public synchronized static void print(String input) {
    		count++;
    		System.out.println(input + "count : "+ count);
    	}
    }
```

* 위와 같이 정적 클래스를 이용하면 객체를 전혀 생성하지 않고 메서드를 사용할 수 있고 인스턴스 메서드를 사용하는 것보다 성능 면에서 우수하다고 볼 수 있다.

2 . **인스턴스를 만드는 메서드에 동기화하는 방법**

하지만 정적 클래스를 사용할 수 없는 경우 또한 존재한다. 인터페이스를 구현하는 경우이다. 인터페이스가 정적 메서드를 가질 수 없기 때문에 이런 경우 정적 클래스를 사용할 수 없다. 예를 들어 모의 객체를 사용하는 경우가 있을 수 있다. 아래의 예를 보자.

```java
    public interface Printer {
    	public void print(String input);
    }
    
    -----------------------
    
    public class RealPrinter implements Printer {
    	private static Printer printer = null;
    
    	private RealPrinter() {
    	}
    
    	public synchronized static Printer getInstance() {
    		if (printer == null)
    			printer = new RealPrinter();
    		return printer;
    	}
    
    	@Override
    	public void print(String input) {
    		System.out.println(input);
    	}
    }
    
    ----------------------
    
    public class UsePrinter {
    	public void doSomething(Printer printer) {
    		printer.print("fakeGet");
    	}
    }
    
    -------------------
    
    public class FakePrinter implements Printer{
    	private String str;
    
    	public void print(String str) {
    		this.str = str;
    	}
    
    	public String get() {
    		return str;
    	}
    }
    
    ---------------------
    
    class UsePrinterTest {
    	@Test
    	void testdoSomething() {
    		FakePrinter fake = new FakePrinter();
    		UsePrinter use = new UsePrinter();
    		use.doSomething(fake);
    		assertThat("fakeGet").isEqualTo(fake.get());
    	}
    }
```

위와 같이 모의 객체를 통해 프린트를 테스트하고자 한다면 Interface를 사용하는 것이 좋을 것이다. 하지만 아까처럼 정적 클래스타입으로 만든다면 위와같은 테스트가 불가능하다. (인터페이스에서 정적 메서드를 사용할 수 없기 때문에)

### 싱글톤의 문제 <a href="#undefined" id="undefined"></a>

* **싱글톤은 프로그램 전체에서 하나의 객체만을 공통으로 사용하고 있기 때문에 각 객체간의 결합도가 높아지고 변경에 유연하게 대처할 수 없다. 싱글톤 객체가 변경되면 이를 참조하고 있는 모든 값들이 변경되어야 하기 때문에**
* **멀티쓰레드 환경에서 대처가 어느정도 가능하지만 고려해야 할 요소가 많아 사용이 어렵고, 프로그램 전반에 걸쳐서 필요한 부분에만 사용한다면 장점이 있다. 하지만 그 포인트를 잡기가 어렵다**

### 결론 <a href="#undefined" id="undefined"></a>

적절한 형태로 싱글톤을 활용하면 좋지만 남용하게 될 여지가 많다. 이 부분에 대해서 어떠한 경우 싱글톤을 사용하면 좋은지는 추가적으로 공부하여 업로드 하도록 하겠습니다. 현재까지의 내용을 기반으로 싱글톤을 사용하면

* 멀티 쓰레드 환경에서의 싱글톤
  * Synchronized를 통해 관리하면 되며 다양한 변화에 대응하기 위해 인터페이스의 형태로 관리하면 좋다.
* 단일쓰레드환경에서 싱글톤
  * 정적 클래스의 형태로 사용하면 된다. (클래스 로딩단계에서 바로 초기화 되도록)
  * 물론 이 경우에서도 테스트를 위한 모의객체를 만들고 혹은 다른 목적으로 사용한다면 멀티쓰레드 환경에서 싱글톤을 사용하듯이 사용하면 된다.

출처: [https://velog.io/@kyle/%EC%9E%90%EB%B0%94-%EC%8B%B1%EA%B8%80%ED%86%A4-%ED%8C%A8%ED%84%B4-Singleton-Pattern](https://velog.io/@kyle/%EC%9E%90%EB%B0%94-%EC%8B%B1%EA%B8%80%ED%86%A4-%ED%8C%A8%ED%84%B4-Singleton-Pattern)&#x20;
