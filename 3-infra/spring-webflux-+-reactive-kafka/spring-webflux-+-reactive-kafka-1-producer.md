# Spring Webflux + Reactive Kafka (1) - Producer



![](https://media.vlpt.us/images/deogicorgi/post/433f771d-a15f-41dc-9282-c6e29631fbec/kafka-producer-consumer-logo.png)

#### 1. 개요 <a href="#1" id="1"></a>

Spring Webflux & Reactive Kafka를 활용하여 API를 통한 프로듀서와 컨슈머를 구성해보려 한다.\
Bloking IO를 사용할때의 개발과는 전혀 달라서 익숙해지는데 꽤나 걸릴 듯 하다. Webflux의 라우팅 방식이 아닌 RestController를 사용하여 구성했다.

기본적인 구성도는 매우 간단하다. 단순히 RestController를 통해 들어온 메시지를 카프카로 전송하고 컨슈머는 카프카에서 메시지를 가져오기만 하는 굉장히 단순한 흐름이다. 단지 이 모든 과정을 논블로킹 I/O로 처리하는것이 핵심이다.

전체 소스 코드 : [Github 링크](https://github.com/deogicorgi/reactor-kafka)

***

#### 2. 카프카 구성 <a href="#2" id="2"></a>

로컬환경이 아닌 실무환경과 비슷하게 외부에 도커를 이용하여 구성하였다. 주키퍼와 브로커는 각 1대씩으로 우선 구성하였고 2대 이상 클러스터는 차후 시간날때 구성하려 한다.

**2-1. docker-compose.yml**

개인 홈서버에 구성하였고 파티션은 1개로 설정했다.

```yaml
---
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://deogicorgi.home:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

***

#### 3. 프로듀서 설정(Producer) <a href="#3-producer" id="3-producer"></a>

Controller를 통해 들어온 메시지를 카프카로 전송하기 위한 Bean 설정 및 옵션 설정이다. 현재 프로듀서와 컨슈머가 독립적인 프로젝트로 구성되어있기에 해당 설정에는 프로듀서용 설정만 해놓은 상태이다.

**3-1. KafkaConfig.java**

Spring Kafka의 설정과는 좀 달라진 부분들이 많다. 매우 기본적인 설정만 구성한 상태이고 이 외에도 레퍼런스를 찾아보면 다양하고 복잡한 설정들을 찾을 수 있다.

공식 레퍼런스 : [공식 레퍼런스 링크](https://projectreactor.io/docs/kafka/1.3.1/reference/index.html#\_what\_s\_new\_in\_reactor\_kafka\_1\_2\_0\_release)

```java
/**
 * Kafka 설정
 */
@Configuration
@RequiredArgsConstructor
public class KafkaConfig {

    private final KafkaProperties properties;

    /******************************************************************
     ************************ Producer Options ************************
     ******************************************************************/
    
    // 기본 설정들로 구성
    @Bean("kafkaSender")
    public KafkaSender<String, Object> kafkaSender() {
        SenderOptions<String, Object> senderOptions = SenderOptions.create(getProducerProps());
        senderOptions.scheduler(Schedulers.parallel());
        senderOptions.closeTimeout(Duration.ofSeconds(5));
        return KafkaSender.create(senderOptions);
    }
    
    // 프로듀서 옵션
    private Map<String, Object> getProducerProps() {
        return new HashMap<>() {{
            put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, properties.getHosts());
            put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
            put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
            put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 1000); // 전송 시간 제한을 1000ms로 설정
        }};
    }
}
```

**3-2. ProduceService.java**

_**KafkaService**_ 클래스에 넘어온 _**AbstractKafkaMessage**_ 클래스는 Controller를 통하여 요청받은 _**@RequestBody**_ 데이터이다. 내부적인 5XX 서버 에러를 리턴해주고 싶지 않기 때문에 _**KafkaProduceResult**_ 라는 클래스를 만들어 자체적으로 처리하도록 구성하려 한다. 즉 요청 측에서는 전송 결과의 StatusCode는 무조건 2XX로 받게 될것이고 내부적으로는 메시지 전송에 실패할 경우 NOSQL이나 기타 방법등을 활용하여 재전송 처리를 할수 있도록 하려 한다.

```java
/**
 * 프로듀싱 서비스
 * Kafka 프로듀싱 전 로직 처리
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class ProduceService {

    private final KafkaService kafkaService;
    private final FailureMessageService failureMessageService;

    public Mono<KafkaProduceResult> produceMessage(AbstractKafkaMessage message) {
        return kafkaService.send(message)
                .map(produceResult -> {
                    log.info("Kafka Sender result : Topic >> [{}], message >> [{}]", produceResult.getTopic(), produceResult.getRequestedMessage());
                    if (produceResult.hasError()) {
                    	failureMessageService.produceFailure();
                        // TODO 카프카 프로듀싱 실패일 경우 처리
                        // ex ) 처리하지못한 요청을 몽고등에 저장 후 재시도, 로깅 등등
                        log.error("Kafka produce error : {}", produceResult.getErrorMessage());
                    }
                    return produceResult;
                });
    }
}
```

**3-3. KafkaService.java**

실제 요청받은 메시지를 카프카로 보내는 코드이다. 실행해보면 100건, 1000건, 10000건이건 간에 싱글스레드로 처리되는데 이 부분을 멀티쓰레드로 돌리고 싶어 구글링을 열심히 해본 결과 Sender의 경우 애초에 싱글스레드로 돌아가도록 구현되어있다고 한다. 옵션에 스케쥴러도 다르게 지정해보고 삽질이란 삽질은 다해봤는데...

```java
/**
 * 카프카 서비스
 * 실제 카프카로 메시지 프로듀싱
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class KafkaService {

    private final KafkaSender<String, Object> producer;

    public Mono<KafkaProduceResult> send(AbstractKafkaMessage message) {

        return producer.createOutbound()
                // 지정된 토픽으로 메시지 전송
                .send(Mono.just(new ProducerRecord<>(message.getTopic(), null, message.getRequestedMessage())))
                .then()
                // 에러 없이 전송이 완료 되었을 경우
                .thenReturn(new KafkaProduceResult(message))
                // 에러가 발생했을 경우
                .onErrorResume(e -> Mono.just(new KafkaProduceResult(message, e)));
    }
}
```

***

#### 4. 그 외 메시지 클래스 <a href="#4" id="4"></a>

요청받은 메시지 매핑 및 전송 처리 결과를 리턴하기 위한 모델 클래스들이다. 위 Kafka관련 클래스들과는 연관 없는 클래스이다. 대충 토이프로젝트의 의도를 보여주기 위함이다.

**4-1. AbstractKafkaMessage.java**

대충 어노테이션을 보면 _**@RequstBody**_ 를 이용해 매핑되는 클래스로 _**KafkaUriMessage**_ 타입과 _**KafkaBodyMessage**_ 타입 있다는 것을 알 수있다. 이는 혹시 전송실패할 경우 두개의 타입을 다르게 처리하려고 나눠놓은 것이다.

```java
/**
 * 카프카 메시지 베이스
 * 프로듀서 내 에러 발생시 처리를 쉽게하기 위해 URI 형태와 Message 형태로 나눔
 */
@Getter
@Setter
@JsonTypeInfo(
        use = JsonTypeInfo.Id.NAME,
        property = "type",
        defaultImpl = KafkaUriProduceMessage.class)
@JsonSubTypes({
        @JsonSubTypes.Type(value = KafkaUriMessage.class, names = {"uri", "Uri", "URI"}),
        @JsonSubTypes.Type(value = KafkaBodyMessage.class, names = {"message", "Message", "MESSAGE"})
})
public abstract class AbstractKafkaMessage {

    // 요청 토픽
    protected String topic;

    // 메시지 타입 (uri , message)
    protected ProduceMessageType type;

    // 요청 시간
    protected LocalDateTime requestedAt;

    public abstract String getRequestedMessage();

}
```

**4-2. KafkaProduceResult.java**

마지막으로 전송결과가 매핑될 클래스이다. 요청 측에서는 해당 클래스의 내용에 따라 전송 성공 & 실패를 확인할 수 있다.

```java
/**
 * 카프카 메시지 전송결과 클래스
 */
@Getter
public class KafkaProduceResult {

    // 메시지 전송 상태 - true : 전송완료, false : 전송실패
    private Boolean status = true;

    // 메시지 전송 토픽
    private String topic;

    // 요청받은 메시지 타입 (uri, message)
    private ProduceMessageType messageType;

    // 요청받은 메시지 - URI 또는 JSON String
    private String requestedMessage;

    // 에러 - 전송과정 중 발생된 에러, 전송완료 일 경우 null
    @JsonIgnore
    private Throwable error = null;

    // 에러 메시지 - 전송과정 중 발생된 에러, 전송완료 일 경우 null
    private String errorMessage = null;

    // 메시지를 요청받은 시간
    private LocalDateTime requestedAt;

    // 메시지를 처리한 시간
    private LocalDateTime producedAt;

    public KafkaProduceResult(AbstractKafkaMessage message) {
        this.setRequestedMessage(message);
    }

    public KafkaProduceResult(AbstractKafkaMessage message, Throwable e) {
        this.setRequestedMessage(message);
        this.status = false;
        this.error = e;
        this.errorMessage = e.getMessage();
        this.producedAt = null;
    }

    public Boolean hasError() {
        return error != null;
    }

    private void setRequestedMessage(AbstractKafkaMessage requestedMessage) {
        this.topic = requestedMessage.getTopic();
        this.messageType = requestedMessage.getType();
        this.requestedMessage = requestedMessage.getRequestedMessage();
        this.producedAt = LocalDateTime.now();
        this.requestedAt = requestedMessage.getRequestedAt();
    }
}
```



출처 : [https://velog.io/@deogicorgi/Spring-Webflux-Reactive-Kafka-1](https://velog.io/@deogicorgi/Spring-Webflux-Reactive-Kafka-1) (굉장히잘 정리해주셨다\~)&#x20;
