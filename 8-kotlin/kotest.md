# Kotest

![](https://velog.velcdn.com/images/plz\_no\_anr/post/4b201cdc-2a53-4157-848f-1d5e0c5b9328/image.png)

****

> 1. 기능을 구현하는 코드 작성
> 2. 그 기능이 정상적으로 작동하는지 AVD나 디바이스에서 빌드하여 기능 확인
> 3. 에러가 발생하면 코드 수정 후 다시 빌드하여 확인

위와 같은 순서를 반복하게 되는데 프로젝트 규모가 작다면 위와 같은 순서로 반복해도 그렇게 크게 차이가 나지 않는다.. 그러나 But 프로젝트의 규모가 커진다면??!!\
앱을 빌드하는 시간 + UI에 입력하여 테스트하는 시간이 더해져 많은 시간을 필요로 할 것이다.

현재 회사에서만 해도 앱 빌드 하는 데만 2\~3분이 소요되고 또 추가한 기능을 테스트하는데 추가적인 시간이 필요하다.

테스트 코드를 작성함으로써 얻는 이점을 살펴보면

> * 잘못된 부분을 빠르게 확인 가능
> * 디버깅 시간 단축
> * 모듈이 의도한대로 동작하고 있는지 확인 가능
> * 클린한 구조의 개발 가능

위와 같은 장점이 많기 때문에 우리는 기능을 테스트할 때 테스트 코드를 작성하도록 해야 한다.

우선 알아야 할게 안드로이드는 디바이스 의존도가 강한 프레임워크라 안드로이드 OS가 설치된 디바이스에서만 동작한다.\
그렇기 때문에 JVM이 실행하는 안드로이드 Unit Test에서 안드로이드의 컴포넌트들을 그대로 사용하거나 객체를 만들 때 문제가 발생한다. (Mocking 같은 방법을 사용하여 가짜 객체를 만들어 테스트)

이와 같은 이유로 최대한 안드로이드 프레임워크에 의존성이 없는 코드로 분리하는 것이 좋다.(DI, MVVM, Clean Architecture등)

***

**안드로이드에는 로컬 단위 테스트(Unit Test)와 계측 테스트(Instrumentation Test) 이 두 가지 테스트가 있다.**

#### 로컬 단위 테스트(Unit Test) <a href="#unit-test" id="unit-test"></a>

* module-name/src/test/java/ 하위에 테스트 코드 작성
* 테스트 속도가 빠르다
* JVM에서 실행되는 테스트
* 안드로이드 프레임워크와 종속성이 없거나 모의 객체를 생성할 수 있는 경우 이 테스트 사용
* JUnit, Mockito, PowerMock, Truth, Robolectric, Kotest

#### 계측 테스트(Instrumentation Test) <a href="#instrumentation-test" id="instrumentation-test"></a>

* module-name/src/androidTest/java/ 하위에 테스트 코드 작성
* 안드로이드 프레임워크에 종속성이 있는 테스트
* 실제 안드로이드 기기나 에뮬레이터에서 실행되는 테스트
* Espresso, UIAnimator, Robotium, Appium, Calabash

**이 중에서 코틀린을 사용하여 Unit Test를 할 수 있는 Kotest라는 라이브러리를 사용해 보려 한다.**

***

### Kotest <a href="#kotest" id="kotest"></a>

**Kotest는 Kotlin을 사용하여 Unit Test를 할 수 있는 라이브러리이다.**

사용 방법으로는 우선

**1. 의존성 추가**

```kotlin
android.testOptions {
   unitTests.all {
      useJUnitPlatform()
   }
}

testImplementation 'io.kotest:kotest-runner-junit5:5.0.3'
```

App단의 Gradle에 위와 같은 의존성을 추가합니다.

**2. 테스트 코드 작성 위치**

\
테스트 코드를 작성하기 위해 프로젝트를 하나 새로 만들었는데 위의 ExampleUnitTest 위치에 테스트할 코틀린 파일을 만들어주면 된다.

**3. 테스트 코드 작성**

우선 Kotest에서는 Spec이라는 이름으로 여러 테스트 스타일을 지원합니다.\
이 중에 가장 간단한건 StringSpec 입니다.\
아래와 같이 작성하는데 테스트 코드는 init 안에 작성하고 그 안에 코드의 Description, 테스트할 코드순으로 작성합니다.

**StringSpec**

```kotlin
class StringTest: StringSpec() { // StringSpec을 상속
    init { // 테스트 코드 작성
        "strings.length should return size of string" {  // 코드 설명 - 테스트에 관여x
            "hello".length shouldBe 5   // hello 문자열의 길이가 5인지 체크
        }
    }
}
```

Kotest에는 StringSpec 말고도 여러 스펙이 있는데 읽는 방식만 다를 뿐 작성하는 테스트 코드는 동일합니다. 이 부분에 대해선 취향

몇 가지 더 예제를 보면

**FunSpec**

```kotlin
class FunSpecTest: FunSpec() { // 함수 형태로 테스트 코드를 작성한다.
    init {
        test("String length should return length of the string") { // 테스트 코드 설명 - description
            "sanggun".length shouldBe 7 // 문자열의 길이를 체크하는 코드
            "hello".length shouldBe 5
        }
    }
}
```

**AnnotationSpec**

```kotlin
class AnnotationTest : AnnotationSpec() { // 어노테이션 형태로 작성, JUnit과 비슷하다.

    @BeforeEach
    fun beforeTest() {
        println("Before each test")
    }

    @Test
    fun test1() {
        1 shouldBe 1
    }

    @Test
    fun test2() {
        3 shouldBe 3
    }
}
```

등등 많은 Spec 이 있습니다. 자세한건 [여기](https://kotest.io/)👈

다음은 **Matcher**입니다.\
Matcher란 테스트 코드를 작성하는데 도와주는 요소들입니다. 예를들어, 아래 코드에서 shouldBe는 동일함을 체크해주는 Matcher입니다.

```kotlin
"sanggun".length shouldBe 7
```

Matcher도 여러 가지가 있는데

```kotlin
class MatcherTest : StringSpec() {
    init {
    -----------------------String Matchers-------------------------
        // 'shouldBe'는 동일함을 체크하는 Matcher
        "hello world" shouldBe haveLength(11) // length가 11이어야 함을 체크
        "hello" should include("ll") // 파라미터가 포함되어 있는지 체크
        "hello" should endWith("lo") // 파라미터가 끝의 포함되는지 체크
        "hello" should match("he...") // 파라미터가 매칭되는지 체크
        "hello".shouldBeLowerCase() // 소문자로 작성되었는지 체크

-----------------------Collection Matchers-------------------------
        val list = emptyList<String>()
        val list2 = listOf("a", "b", "c")
        val map = mapOf<String, String>(Pair("a", "1"))

        list should beEmpty() // 원소가 비었는지 체크 합니다.
        list2 shouldBe sorted<String>() // 해당 자료형이 정렬 되었는지 체크
        map should contain("a", "1") // 해당 원소가 포함되었는지 체크
        map should haveKey("a") // 해당 key가 포함되었는지 체크
        map should haveValue("1") // 해당 value가 포함되었는지 체크
    }
}
```

위와 같이 많은 Matcher들이 존재합니다.

\
