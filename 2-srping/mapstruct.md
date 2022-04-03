#  MapStruct

![](https://media.vlpt.us/images/cham/post/ce0a4581-7a5f-44bf-b154-7c84a52e0aec/Spring.png)

## 0. 들어가기 전에 <a href="#0" id="0"></a>

***

MapStruct는 Entity와 Dto간의 매핑을 지원하는 라이브러리다.

Entity와 Dto간의 매핑을 위해 getter/setter를 남발하며 직접 구현하는 것을 지원하는 라이브러리는 크게 ModelMapper와 MpaStruct가 있다.

주로 쓰이는 ModelMapper와 비교했을때, MapStruct는 컴파일시 미리 생성된 구현체를 통해 Mapping하기 때문에 속도적인 측면에서 이점이 있어 채택했다.

프로젝트에 MapStruct를 사용해 간단한 사용기를 기록한다.

* 개발환경
  * IntelliJ 2020.2
  * Java 8
  * SpringBoot 2.4.2
  * Gradle 6.8.2

\
\
\
\


## 1. 환경설정 <a href="#1" id="1"></a>

***

Gradle에 다음과 같이 dependencies를 설정한다

\
\


```java
        implementation 'org.mapstruct:mapstruct:1.4.1.Final'
        implementation 'org.projectlombok:lombok-mapstruct-binding:0.2.0'
        implementation "org.projectlombok:lombok:1.18.16"

        annotationProcessor "org.mapstruct:mapstruct-processor:1.4.1.Final"
        annotationProcessor "org.projectlombok:lombok-mapstruct-binding:0.2.0"
        annotationProcessor "org.projectlombok:lombok:1.18.16"
```

\
\
\
\


## 2. MapStruct 적용 <a href="#2-mapstruct" id="2-mapstruct"></a>

***

MapStruct를 적용하기 위해 Generic을 사용해 미리 interface를 만든다

\
\
\
**GenericMapper interface**

```java
public interface GenericMapper<D, E> {

    D toDto(E e);
    E toEntity(D d);

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateFromDto(D dto, @MappingTarget E entity);
}
```

> updateFromDto()는 기존 생성되어있는 Entity를 업데이트하고 싶을 때 null이 아닌 값만 업데이트할 수 있다.

* `@MappingTarget` : 변환하여 객체를 return하는 것이 아닌 인자로 받아 업데이트할 target을 설정한다.
* `@BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)`
  * Source의 필드가 null일 때 정책으로 null인 값은 무시한다.

이제 이 GenericMapper을 상속만 받은 뒤 `@Mapper`만 선언해주면 된다.

\
\
\


**CodingRoomMapper interface**

```java
@Mapper(componentModel = "spring")
public interface CodingRoomMapper extends GenericMapper<CodingRoomDto, CodingRoom> {


}
```

* `@Mapper` : MapStruct Code Generator가 해당 인터페이스의 구현체를 생성해준다.
* `componentModel = "spring"` : spring에 맞게 bean으로 등록해준다

\
\
\


**CodingRoomMapperImpl class**

\


```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2021-05-28T02:50:51+0900",
    comments = "version: 1.4.1.Final, compiler: javac, environment: Java 1.8.0_231 (Oracle Corporation)"
)
@Component
public class CodingRoomMapperImpl implements CodingRoomMapper {

    @Override
    public CodingRoomDto toDto(CodingRoom arg0) {
        if ( arg0 == null ) {
            return null;
        }

        CodingRoomDto codingRoomDto = new CodingRoomDto();

        if ( arg0.getKey() != null ) {
            codingRoomDto.setKey( String.valueOf( arg0.getKey() ) );
        }
        codingRoomDto.setTitle( arg0.getTitle() );
        codingRoomDto.setIntro( arg0.getIntro() );
        codingRoomDto.setPassword( arg0.getPassword() );
        codingRoomDto.setMaxUser( arg0.getMaxUser() );
        codingRoomDto.setRamdomKey( arg0.getRamdomKey() );
        codingRoomDto.setCreatedAt( arg0.getCreatedAt() );
        codingRoomDto.setCodingJoinUsers( codingJoinUserListToCodingJoinUserDtoList( arg0.getCodingJoinUsers() ) );
        codingRoomDto.setCodingTests( codingTestListToCodingTestDtoList( arg0.getCodingTests() ) );

        return codingRoomDto;
    }

    @Override
    public CodingRoom toEntity(CodingRoomDto arg0) {
        if ( arg0 == null ) {
            return null;
        }

        CodingRoom codingRoom = new CodingRoom();

        if ( arg0.getKey() != null ) {
            codingRoom.setKey( Long.parseLong( arg0.getKey() ) );
        }
        codingRoom.setTitle( arg0.getTitle() );
        codingRoom.setIntro( arg0.getIntro() );
        codingRoom.setPassword( arg0.getPassword() );
        codingRoom.setMaxUser( arg0.getMaxUser() );
        codingRoom.setRamdomKey( arg0.getRamdomKey() );
        codingRoom.setCreatedAt( arg0.getCreatedAt() );
        codingRoom.setCodingJoinUsers( codingJoinUserDtoListToCodingJoinUserList( arg0.getCodingJoinUsers() ) );
        codingRoom.setCodingTests( codingTestDtoListToCodingTestList( arg0.getCodingTests() ) );

        return codingRoom;
    }

...
}
```

> build한다면 @Mapper가 붙은 interface에 대한 구현체가 자동 생성되는 것을 볼 수 있다.

\
\


**CodingRoomService class**

\


```java
@Service
@RequiredArgsConstructor
public class CodingRoomService {

    private final CodingRoomMapper codingRoomMapper;
    ...
        
    CodingRoomDto codingRoomDto =  codingRoomMapper.toDto(codingRoom);
    ...        
    }
    
```

&#x20; 각 구현체를 받아와 사용하면 된다.

\
\
\
\


***

참조 사이트

* [https://mapstruct.org/documentation/stable/reference/html/](https://mapstruct.org/documentation/stable/reference/html/)
* [https://www.skyer9.pe.kr/wordpress/?p=1596](https://www.skyer9.pe.kr/wordpress/?p=1596)

[![profile](https://media.vlpt.us/images/cham/profile/fa1abd6b-542f-4663-af6f-6143c9c440fa/images.jpg?w=240)](https://velog.io/@cham)\
