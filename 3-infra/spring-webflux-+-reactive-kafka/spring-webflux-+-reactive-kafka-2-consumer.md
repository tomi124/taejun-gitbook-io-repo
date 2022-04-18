# Spring Webflux + Reactive Kafka (2) - Consumer

![](https://media.vlpt.us/images/deogicorgi/post/6172fa77-5b7f-472b-bd0f-21f190604fbd/kafka-producer-consumer-logo.png)

#### 1. 개요 <a href="#1" id="1"></a>

이번엔 컨슈머를 구성한다. 이번 역시 _**Reactive Kafka**_ 를 통하여 컨슈머를 구성할 예정이고 앞서 구성한 프로듀서와 연동하여 실제 메시지를 받는것까지 구현해보려 한다.

전체 소스코드 : [Github 링크](https://github.com/deogicorgi/reactor-kafka)

***

#### 2. 카프카 토픽 구성내용 <a href="#2" id="2"></a>

현재 개인 서버 카프카에 구성된 토픽은 총 2개로 _**deogicorgi-message**_, _**deogicorgi-uri**_ 가 존재한다. 각 토픽의 파티션은 1개씩 구성하였으며 그렇기 때문에 하나의 토픽에는 하나의 컨슈머만 붙일 수 있는 상태이다. (파티션과 컨슈머의 관계는 더 공부 해봐야 한다.)

```bash
[appuser@d97c9a0c3c7e ~]$ kafka-topics --list --bootstrap-server localhost:9092
__consumer_offsets
deogicorgi-message
deogicorgi-uri
[appuser@d97c9a0c3c7e ~]$
```

정상적으로 카프카 토픽이 존재함을 확인할 수 있다. (도커 컨테이너 내에서 수행한 커맨드이다.)

***

#### 3. 컨슈머 설정(Consumer) <a href="#3-consumer" id="3-consumer"></a>

우선 메시지를 받는 기능만 구성하기 때문에 간단하게 설정하였다. 차후 메시지를 다시 프로듀싱하거나 RDBMS 또는 NoSQL에 저장하는 로직 역시 추가할 예정이다. 현재 계획하는 전체적인 플로우는 아래와 같다.

> 프로듀서 - 컨슈머 - 저장매체 or 프로듀서 (이 모든과정이 논블로킹으로 구성되어야 한다)

**3-1. application.yml**

카프카가 설치된 개인서버 주소 및 메시지를 받을 토픽, 컨슈머의 그룹 아이디 등을 설정한다.

```yaml
kafka:
  hosts: deogicorgi.home:29092
  receiver :
    uri:
      name : deogicorgiUri
      topic : deogicorgi-uri
      groupId : deogicorgi-uri-1
    message:
      name : deogicorgiMessage
      topic : deogicorgi-message
      groupId : deogicorgi-message-1
```

**3-2. KafkaProperties.java**

위 의 _**application.yml**_ 과 매핑될 클래스이다. yml 파일 내 receiver 2개를 하나의 맵으로 받도록 구성했다.

```java
@Getter
@Setter
@Component
@EnableConfigurationProperties
@ConfigurationProperties("kafka")
public class KafkaProperties {

    // 카프카 호스트
    private String hosts;

    // 리시버 프로퍼티 맵
    private Map<String, KafkaReceiverProperty> receiver = new HashMap<>();

    public void setReceiver(Map<String, KafkaReceiverProperty> receivers) {
        this.receiver = receivers;
    }

    public Optional<Map.Entry<String, KafkaReceiverProperty>> getProperty(String key) {
        return this.receiver.entrySet()
                .stream().filter(entry -> entry.getValue().getName().equals(key))
                .findFirst();
    }
}
```

**3-3. KafkaReceiverProperty.java**

위 의 _**KafkaProperties**_ 에서 리시버 맵에 사용되는 모델클래스이다.

```java
/**
 * 카프카 리시버 설정 프로퍼티
 */
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class KafkaReceiverProperty {

    // 리시버 이름
    private String name;

    // 담당할 토픽
    private String topic;

    // 리시버 그룹 아이디
    private String groupId;
}
```

**3-4. KafkaConfig.java**

두개의 토픽을 각각 컨슈밍하는 리시버들를 빈으로 등록했다. 각각의 빈들은 설정된 토픽의 메시지만 담당하여 받게될 것이고 각 토픽들의 파티션은 1개로 구성하였기 때문에 싱글 스레드로 동작하게 될것이다. 여기서는 리시버당 토픽을 1개씩만 지정하였지만 하나의 리시버에 여러개의 토픽을 지정 할 수 도있다. 그렇게 되면 등록된 여러개의 토픽을 싱글 스레드로 받게된다.

```java
@Configuration
@RequiredArgsConstructor
public class KafkaConfig {

    private final KafkaProperties properties;

    // deogicorgi-uri 리시버
    @Bean("uriMessageReceiver")
    public KafkaReceiver<Integer, String> uriMessageReceiver() throws Exception {
        Map.Entry<String, KafkaReceiverProperty> deogicorgiUri = properties.getProperty("deogicorgiUri").orElse(null);

        if (ObjectUtils.isEmpty(deogicorgiUri)) {
            throw new Exception("property is null");
        }

        KafkaReceiverProperty property = deogicorgiUri.getValue();

        ReceiverOptions<Integer, String> receiverOptions =
                ReceiverOptions.<Integer, String>create(getConsumerProps(property))
                        .subscription(Collections.singleton(property.getTopic()));

        return KafkaReceiver.create(receiverOptions);
    }

    // deogicorgi-message 리시버
    @Bean("messageReceiver")
    public KafkaReceiver<Integer, String> messageReceiver() throws Exception {
        Map.Entry<String, KafkaReceiverProperty> deogicorgiUri = properties.getProperty("deogicorgiMessage").orElse(null);

        if (ObjectUtils.isEmpty(deogicorgiUri)) {
            throw new Exception("property is null");
        }

        KafkaReceiverProperty property = deogicorgiUri.getValue();

        ReceiverOptions<Integer, String> receiverOptions =
                ReceiverOptions.<Integer, String>create(getConsumerProps(property))
                        .subscription(Collections.singleton(property.getTopic()));

        return KafkaReceiver.create(receiverOptions);
    }

    /******************************************************************
     ************************ Consumer Options ************************
     ******************************************************************/

    // 컨슈머 옵션
    private Map<String, Object> getConsumerProps(KafkaReceiverProperty property) {
        return new HashMap<>() {{
            put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, properties.getHosts());
            put(ConsumerConfig.GROUP_ID_CONFIG, property.getGroupId());
            put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
            put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
            put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 1000);
        }};
    }
}
```

**3-5. KafkaMessageReceiver.java**

빈으로 등록된 리시버들을 제공받아 실제로 컨슈밍하는 코드이다. _**KafkaMessageReceiver**_ 는 빈으로 등록 될 때 정의된 모든 리시버를 받아와 컨슈밍을 시작한다.\
스프링 빈의 라이프사이클을 알고있다면 해당 클래스가 생성되고 제거될때 수행되는 메서드들을 오버라이딩 하여 전처리 및 후처리를 진행 할 수 있다.

```java
/**
 * 카프카 리시버
 */
@Slf4j
@Component
public class KafkaMessageReceiver {

    /**
     * KafkaMessageReceiver가 생성될 때 모든 카프카 리시버 시작
     */
    public KafkaMessageReceiver(List<KafkaReceiver<Integer, String>> kafkaReceivers) {
        for (KafkaReceiver<Integer, String> receiver : kafkaReceivers) {
            this.start(receiver);
        }
    }

    public void start(KafkaReceiver<Integer, String> receiver) {
        receiver.receive().subscribe(record -> {
            log.info("Kafka Reciever result : Topic >> [{}], message >> [{}], Offset >> [{}]", record.topic(), record.value(), record.receiverOffset());
        });
    }
}
```

**3-6. 로그**

Producer의 경우 모든 메시지를 싱글 스레드로 처리 한다. 각각 의 토픽에 하나의 메시지를 보냈을 경우 _**kafka-producer-network-thread**_ 라는 이름의 스레드를 통하여 처리된다. 해당 스레드는 기본값으로, 다른 스레드풀을 사용하고 싶다면 설정에서 사용하고싶은 스레드를 등록해주면 된다.\


![](https://media.vlpt.us/images/deogicorgi/post/aa64c0ce-349e-4092-9b31-854b5bdeda2e/sender.png)

Consumer의 경우 각각의 리시버가 다른 스레드를 사용하는 것을 확인할 수 있다. _**reactive-kafka-\***_ 로 시작하는 스레드를 각각 사용하고 있으며 현재 각 토픽의 파티션이 1개씩이므로 해당 스레드만 계속 사용하여 처리할 것이다. 이후 각 토픽의 파티션을 늘리면 각 리시버들은 등록된 쓰레드풀을 통하여 멀티쓰레드로 메시지를 처리하게 될것이다.\


![](https://media.vlpt.us/images/deogicorgi/post/e7a636a2-db74-4182-a8e4-ed42ea4aa654/receiver.png)

출처 : [https://velog.io/@deogicorgi/Spring-Webflux-Reactive-Kafka-2-Consumer](https://velog.io/@deogicorgi/Spring-Webflux-Reactive-Kafka-2-Consumer) (굉장히 잘 정리해주셨다)&#x20;
