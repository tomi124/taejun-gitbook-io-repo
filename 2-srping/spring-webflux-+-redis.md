# Spring WebFlux + Redis

&#x20;

**1. redis-reactive, embedded redis 의존성 추가**

```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-redis-reactive'
	implementation 'it.ozimov:embedded-redis:0.7.2'
    ...
}
```

&#x20;

**2. redis의 문자열의 key value를 사용하도록 Bean 등록**

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.connection.ReactiveRedisConnectionFactory;
import org.springframework.data.redis.core.ReactiveRedisOperations;
import org.springframework.data.redis.core.ReactiveRedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class BasicRedisConfig {

    @Primary
    @Bean
    ReactiveRedisOperations<String, String> redisOperations(ReactiveRedisConnectionFactory factory) {
        RedisSerializer<String> serializer = new StringRedisSerializer();
        RedisSerializationContext<String, String> serializationContext = RedisSerializationContext
                .<String, String>newSerializationContext()
                .key(serializer)
                .value(serializer)
                .hashKey(serializer)
                .hashValue(serializer)
                .build();

        return new ReactiveRedisTemplate<>(factory, serializationContext);
    }
}
```

&#x20;

**3. test 및 로컬 test를 위한 embedded redis 설정**

```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import redis.embedded.RedisServer;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Configuration
public class EmbeddedRedisConfig {

    @Value("${spring.redis.port}")
    private int redisPort;

    private RedisServer redisServer;

    @PostConstruct
    public void redisServer() {
        redisServer = new RedisServer(redisPort);
        redisServer.start();
    }

    @PreDestroy
    public void stopRedis() {
        redisServer.stop();
    }
}
```

&#x20;

**4. handler 작성**

&#x20;\- loadData(): 10만개의 데이터 로드 메서드

&#x20;\- findReactorList(): 비동기, 논블로킹 방식의 메서드

&#x20;\- findNormalList(): 동기, 블로킹 방식의 메서드

```
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.UUID;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

@Component
@RequiredArgsConstructor
public class BasicHandler {

    private final ReactiveRedisConnectionFactory factory;
    private final ReactiveRedisOperations<String, String> reactiveRedisOperations;
    private final RedisTemplate<String , String> stringStringRedisTemplate;
    private static final AtomicInteger count = new AtomicInteger(0);

    public void loadData() {
        List<String> data = new ArrayList<>();
        IntStream.range(0, 100000).forEach(i -> data.add(UUID.randomUUID().toString()));

        Flux<String> stringFlux = Flux.fromIterable(data);

        factory.getReactiveConnection()
                .serverCommands()
                .flushAll()
                .thenMany(stringFlux.flatMap(uid -> reactiveRedisOperations.opsForValue()
                        .set(String.valueOf(count.getAndAdd(1)), uid)))
                .subscribe();
    }

    public Flux<String> findReactorList() {
        return reactiveRedisOperations
                .keys("*")
                .flatMap(key -> reactiveRedisOperations.opsForValue().get(key));
    }

    public Flux<String> findNormalList() {
        return Flux.fromIterable(Objects.requireNonNull(stringStringRedisTemplate.keys("*"))
                .stream()
                .map(key -> stringStringRedisTemplate.opsForValue().get(key))
                .collect(Collectors.toList()));
    }
}
```

&#x20;

**5. router 등록**

&#x20;\- 등록시 bean이름이 다른 router와 겹치지 않게 조심해야함. 동일 이름의 bean 2개 만들려고하면서 실패하게 됨

```
import com.webflux.study.handler.BasicHandler;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.web.reactive.function.server.ServerResponse;

@Configuration
@RequiredArgsConstructor
public class BasicRouter {

    private final BasicHandler basicHandler;

    @Bean
    public RouterFunction<ServerResponse> basicRoute() {
        return RouterFunctions.route()
                .GET("/reactive-list", serverRequest -> ServerResponse.ok().contentType(MediaType.TEXT_EVENT_STREAM)
                        .body(basicHandler.findReactorList(), String.class))
                .GET("/normal-list", serverRequest -> ServerResponse.ok().contentType(MediaType.TEXT_EVENT_STREAM)
                        .body(basicHandler.findNormalList(), String.class))
                .GET("/load", serverRequest -> { basicHandler.loadData(); return ServerResponse.ok()
                        .body(BodyInserters.fromValue("Load Data Completed")); })
                .build();
    }
}
```

&#x20;

출처 : [https://wellbell.tistory.com/238](https://wellbell.tistory.com/238)
