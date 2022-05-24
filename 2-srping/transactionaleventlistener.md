# TransactionalEventListener

### @TransactionalEventListener

**Event를 사용할 때 기본적으로 사용하는 @EventListener는 event를 publishing 하는 코드 시점에 바로 publishing**합니다. 그런데 우리는 event를 퍼블리싱 할때는 대부분 메인 작업이 아닌 서브의 작업이 많고 비동기로 진행해도 되는 경우도 많습니다. 다른 도메인 로직인 경우도 있죠. 이럴 경우 조금 애매해지기도 합니다.

**아래의 예제로 상황을 보도록 하겠습니다.** 아래코드는 @Transactional로 메서드를 하나의 트랜잭션으로 묶어두었습니다. 1번과 2번이 정상적으로 마무리되고 3번이 발생하는 도중에 예외처리가 발생하면 어떻게 될까요 ? 3번은 실패했으며 1번도 같은 트랜잭션으로 묶여 있기때문에 실패할 것입니다. 하지만 2번은 rollback이 이루어지지 않기때문에 **결과적으로 불일치가 발생**할 수 밖에 없게 되는 것입니다.

```
@Transactional
public void function() {

    reviewRepository.save() // 1. A 저장

    applicationEventPublisher.publishEvent(); // 2. A에 의한 이벤트 발생

    userRepository.save() // 3. B 저장

}
```

이러한 문제를 해결하기 위해서 `@TransactionEventListener`가 나왔습니다. `@TransactionEventListener`는 Event의 실질적인 발생을 트랜잭션의 종료를 기준으로 삼는것입니다.

### @TransactionalEventListener 옵션

`@TransactionalEventListener`을 이용하면 트랜잭션의 어떤 타이밍에 이벤트를 발생시킬 지 정할 수 있습니다. 옵션을 사용하는 방법은 `TransactionPhase`을 이용하는 것이며 아래와 같은 옵션을 사용할 수 있습니다.

* **AFTER\_COMMIT (기본값)** - 트랜잭션이 성공적으로 마무리(commit)됬을 때 이벤트 실행
* AFTER\_ROLLBACK – 트랜잭션이 rollback 됬을 때 이벤트 실행
* AFTER\_COMPLETION – 트랜잭션이 마무리 됬을 때(commit or rollback) 이벤트 실행
* BEFORE\_COMMIT - 트랜잭션의 커밋 전에 이벤트 실행

### 실습 사전 준비

그렇다면 사용하면서 알아보도록 하겠습니다. 오늘 사용할 실습 예제코드는 아래와 같습니다. 이전 코드에서 Model은 동일하며 Service와 Listener의 코드는 일부 변경된 부분이 있으니 확인해주시기 바랍니다.

#### Service 코드

아래는 Service 코드입니다. `@Transactional`을 이용하여 트랜잭션을 사용하며 review와 user를 DB에 접근하면서 그 사이에 이벤트를 호출하는 것을 알 수 있습니다.

```
@Slf4j
@Component
public class EventTestService {

    private final UserRepositoryV1 userRepositoryV1;

    private final ReviewRepositoryV1 reviewRepositoryV1;

    private final ApplicationEventPublisher applicationEventPublisher;

    public EventTestService(UserRepositoryV1 userRepositoryV1,
                            ReviewRepositoryV1 reviewRepositoryV1,
                            ApplicationEventPublisher applicationEventPublisher) {
        this.userRepositoryV1 = userRepositoryV1;
        this.reviewRepositoryV1 = reviewRepositoryV1;
        this.applicationEventPublisher = applicationEventPublisher;
    }

    @Transactional // 트랜잭션 사용
    public void publishTransactionalCustomEvent() {

        Review review = reviewRepositoryV1.findById(1L)
                .orElseThrow();

        User user = userRepositoryV1.findById(review.getUserId())
                .orElseThrow();

        reviewRepositoryV1.save(review);


        applicationEventPublisher.publishEvent(new DomainEvent("karol", 15));

        userRepositoryV1.save(user);
    }
}
```

#### Listener 코드

아래는 Listener 코드입니다. `@EventListener` 대신 `@TransactionalEventListener`을 사용합니다.

```
@Slf4j
@Component
public class TransactionalEventTestListener {

    @TransactionalEventListener
    public void handleContextStart2(DomainEvent cse) {
        log.info("name = " + cse.getName() + ", age = " + cse.getAge());
        log.info("TransactionalEventListener 이벤트 마무리");
    }
}
```

#### Model 코드

아래는 모델 코드로 이전 코드와 달라진점은 없습니다.

```
public class DomainEvent {

    private final String name;
    private final int age;

    public DomainEvent(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

### 실습

위의 코드를 가지고 실습을 진행해볻록 하겠습니다.

#### 기본적인 상황

위 코드를 기본적으로 실행해보겠습니다. 실행 순서를 확인하기 위해서 log를 심어놓도록 하겠습니다.

```
@Transactional
public void publishTransactionalCustomEvent(Boolean isException) {
    log.info("메서드 시작");
    Review review = reviewRepositoryV1.findById(1L)
            .orElseThrow();

    User user = userRepositoryV1.findById(review.getUserId())
            .orElseThrow();

    reviewRepositoryV1.save(review);
    log.info("reviewRepositoryV1 저장");

    applicationEventPublisher.publishEvent(new DomainEvent("karol", 15));
    log.info("이벤트 발생 요청");

    if (isException) {
        log.info("에러 발생");
        throw new RuntimeException("exception");
    }

    userRepositoryV1.save(user);
    log.info("userRepositoryV1 저장");
    log.info("메서드 종료");
}
```

결과를 보도록 하겠습니다. 로그가 출력된 순서를 보면 `publishEvent`의 순간 Event가 실행되지 않는 것을 알 수 있습니다. **Event가 실행되는 순간은 바로 메서드가 종료되서 트랜잭션이 종료되는 순간이라는 사실을 확인할 수 있었습니다.**

```
2021-08-22 22:34:28.268  INFO 13835 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 시작
2021-08-22 22:34:28.355  INFO 13835 --- [    Test worker] c.p.a.api.game.service.EventTestService  : reviewRepositoryV1 저장
2021-08-22 22:34:28.356  INFO 13835 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 이벤트 발생 요청
2021-08-22 22:34:28.359  INFO 13835 --- [    Test worker] c.p.a.api.game.service.EventTestService  : userRepositoryV1 저장
2021-08-22 22:34:28.359  INFO 13835 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 종료
2021-08-22 22:34:28.378  INFO 13835 --- [    Test worker] p.a.a.g.s.TransactionalEventTestListener : TransactionalEventListener 이벤트 마무리
```

#### 에러 발생

에러가 발생한다면 어떻게 될까요 ? 위 메서드에서 `isException`을 true로 파라미터를하여 메서드를 실행시키면 아래와 같은 로그가 찍힙니다. **에러가 발생하기 때문에 트랜잭션은 롤백되며 Event도 실행되지 않는것을 확인할 수 있었습니다.**

```
2021-08-22 22:36:15.959  INFO 13944 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 시작
2021-08-22 22:36:16.048  INFO 13944 --- [    Test worker] c.p.a.api.game.service.EventTestService  : reviewRepositoryV1 저장
2021-08-22 22:36:16.049  INFO 13944 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 이벤트 발생 요청
2021-08-22 22:36:16.050  INFO 13944 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 에러 발생
java.lang.RuntimeException: exception
```

#### 옵션 변경

모든 옵션을 하나씩 테스트 해보도록 하겠습니다. 테스트할 `TransactionPhase`를 하나씩 만들고 테스트해보겠습니다. 먼저 성공의 경우 3가지 타입의 Event가 발생하는것을 확인하였습니다. 그 이벤트들은 아래와 같습니다.

```
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
```

```
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION)
```

```
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION)
```

순서적으로는 아래와 같이 노출되었습니다. `AFTER_COMMIT`과 `AFTER_COMPLETION`은 ORDER에 따라 달라질 수 있습니다.

```
2021-08-22 22:49:03.816  INFO 14681 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 시작
2021-08-22 22:49:03.891  INFO 14681 --- [    Test worker] c.p.a.api.game.service.EventTestService  : reviewRepositoryV1 저장
2021-08-22 22:49:03.893  INFO 14681 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 이벤트 발생 요청
2021-08-22 22:49:03.895  INFO 14681 --- [    Test worker] c.p.a.api.game.service.EventTestService  : userRepositoryV1 저장
2021-08-22 22:49:03.895  INFO 14681 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 종료
2021-08-22 22:49:03.896  INFO 14681 --- [    Test worker] p.a.a.g.s.TransactionalEventTestListener : TransactionalEventListener BEFORE_COMMIT
2021-08-22 22:49:03.909  INFO 14681 --- [    Test worker] p.a.a.g.s.TransactionalEventTestListener : TransactionalEventListener AFTER_COMMIT
2021-08-22 22:49:03.909  INFO 14681 --- [    Test worker] p.a.a.g.s.TransactionalEventTestListener : TransactionalEventListener AFTER_COMPLETION
```

실패의 경우는 아래와 같은 Event가 발생하였으며 순서는 아래와 같았습니다. 이 순서역시 `ORDER`에 따라 달라질 수 있습니다.

```
@TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
```

```
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION)
```

```
2021-08-22 22:51:38.000  INFO 14833 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 시작
2021-08-22 22:51:38.073  INFO 14833 --- [    Test worker] c.p.a.api.game.service.EventTestService  : reviewRepositoryV1 저장
2021-08-22 22:51:38.075  INFO 14833 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 이벤트 발생 요청
2021-08-22 22:51:38.075  INFO 14833 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 에러 발생
2021-08-22 22:51:38.082  INFO 14833 --- [    Test worker] p.a.a.g.s.TransactionalEventTestListener : TransactionalEventListener AFTER_COMPLETION
2021-08-22 22:51:38.083  INFO 14833 --- [    Test worker] p.a.a.g.s.TransactionalEventTestListener : TransactionalEventListener AFTER_ROLLBACK
```

#### @Transactional 없을 경우

**마지막으로는 `@Transactional`이 없을경우를 테스트 해보았습니다. `@TransactionalEventListener`는 트랜잭션에 의존하여 발생합니다. 따라서 `@Trasnactional`이 없을때는 Event가 발생하지 않는것을 확인할 수 있었습니다.**

```
2021-08-22 22:38:18.191  INFO 14087 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 시작
2021-08-22 22:38:18.329  INFO 14087 --- [    Test worker] c.p.a.api.game.service.EventTestService  : reviewRepositoryV1 저장
2021-08-22 22:38:18.330  INFO 14087 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 이벤트 발생 요청
2021-08-22 22:38:18.340  INFO 14087 --- [    Test worker] c.p.a.api.game.service.EventTestService  : userRepositoryV1 저장
2021-08-22 22:38:18.340  INFO 14087 --- [    Test worker] c.p.a.api.game.service.EventTestService  : 메서드 종료
```

### 마무리

오늘은 이렇게 TrnasactionalEventListener에 대해서 이론적인 부분에 대해서 알아보았습니다.

그리고 실습을 통해 어떤 타이밍에 이벤트가 호출되는지 알아보았습니다.

감사합니다.

### 참조

[baeldung\_spring-events#transaction-bound-events](https://www.baeldung.com/spring-events#transaction-bound-events)

[baeldung\_transaction-configuration-with-jpa-and-spring](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)

[stackoverflow\_transactionaleventlistener-doesnt-works-where-as-eventlistener-works-like-cha](https://stackoverflow.com/questions/51097916/transactionaleventlistener-doesnt-works-where-as-eventlistener-works-like-cha)
