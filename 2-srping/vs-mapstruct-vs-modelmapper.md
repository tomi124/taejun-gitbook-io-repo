# 자바 코드 매핑 vs MapStruct vs ModelMapper

해당 글은 MapStruct Library를 실무에서 사용하기 이전에 간단하게 기록했던 예제와 장, 단점을 옮겨온 글입니다.

&#x20;

&#x20;

![](https://blog.kakaocdn.net/dn/RRBsY/btqZct4uvfW/L9PdYeeiwM2LNbdYFDK8Gk/img.png)

&#x20;

#### &#x20;

#### **자바** **코드로 매핑하기**

어떠한 라이브러리를 사용하지 않고 직접 객체 상태 간의 매핑 로직을 구현하는 방식은 약간의 수고스러움은 있으나 ModelMapper와 같이 Reflection 기반의 라이브러리보다 안전하다.&#x20;

&#x20;

\
**entity, dto**

```
@ToString
@Getter
@NoArgsConstructor
public class SampleEntity {

    private Long id;
    private String name;
    private String email;
    private Long age;
    private List<String> sampleInfo;
    private String value;

    @Builder
    public SampleEntity (Long id, String name, String email, Long age, List<String> samples, String value) {
    	this.id = id;
        this.name = name;
        this.email = email;
        this.age = age;
        this.sampleInfo = samples;
        this.value = value;
    }
}

@ToString
@Getter
@NoArgsConstructor
public class SampleDto {

    private final String name;
    private final String email;
    private final List<String> infos;
    private final Long age;

    @Builder
    public SampleDto(String name, String email, List<String> sampleinfo, Long age) {
    	this.name = name;
        this.email = email;
        this.infos = sampleinfo;
        this.age = age;
    }

    public SampleEntity toEntity() {
        return SampleEntity.builder()
                .name("lob")
                .email("...@test")
                .age(20L)
                .value("value")
                .sampleInfo(new ArrayList<>(Collections.singleton("aaa")))
                .build();
    }

    public static SampleDto toDto(SampleEntity entity) {
        return SampleDto.builder()
                .name(entity.getName())
                .email(entity.getEmail())
                .infos(entity.getSampleInfo())
                .age(entity.getAge())
                .build();
    }
}
```

_기존 코드 방식은 모든 필드에 대해서 일일이 매핑을 진행하여야 한다._

```
public void dtoToEntity() {
       SampleDto dto = SampleDto.builder()
                .name("dto")
                .email("...@hello")
                .age(20L)
                .infos(new ArrayList<>(Collections.singleton("aaa")))
                .build();

        SampleEntity entity = dto.toEntity();
        System.out.println(entity);
}

public void EntityToDto() {
        SampleEntity entity = SampleEntity.builder()
                .id(1L)
                .name("lob")
                .email("...@test")
                .age(20L)
                .value("value")
                .sampleInfo(new ArrayList<>(Collections.singleton("aaa")))
                .build();

        SampleDto dto = SampleDto.toDto(entity);
        System.out.println(dto);
}
```

```
SampleEntity(id=null, name=lob, email=...@test, age=20, sampleInfo=[aaa], value=value)
SampleDto(name=lob, email=...@test, infos=[aaa], age=20)
```

\
**해당 방식의 장, 단점**\
\
**장점**

* 객체 변환을 위한 별도의 과정을 거치지 않고 메서드 호출만 하기 때문에 성능에 대한 영향이 없다.
* 이름이 다른 필드 간의 매핑도 그저 Getter 등을 작성하여 올바르게 조합하기만 하면 된다.
* 매핑하는 필드 타입이 다른 경우에 컴파일 타임에 이를 식별할 수 있다.&#x20;

**단점**

* 객체의 필드 명 변경이나 추가 시 매핑하는 코드 부분도 같이 수정하여야 한다. (변경 지점이 늘어날 수 있다.)
* 필드가 너무 많거나 조합하는 형태의 데이터가 많다면 흔히 말하는 휴먼 에러가 발생할 수 있다. (다른 필드와의 매핑이나 데이터 누락 등)

&#x20;

&#x20;

&#x20;

#### **MapStruct**

***

간결한 객체 간의 변환을 위해 사용되는 라이브러리이다. 컴파일 시점에 매핑 정보를 생성하고 이를 사용하여 객체를 매핑하기에 보일러 플레이트 코드를 제거하고 깔끔한 코드를 유지하게 된다.\
\
Annotation processor를 이용해 메서드 인자와 반환할 값이 될 객체에 필요한 메서드를 호출(builder, getter)하여 자동으로 객체 간의 매핑을 제공한다.  \
\
\
**기능 지원**

* 기본 값과 상수에 대한 매핑을 지원한다.
* 다른 필드 타입을 변환하여 매핑하는 기능을 제공한다.
* @Mapping 어노테이션의 expression에서 문자열로 표현식을 줄 수 있다. _여러 개의 속성을 하나의 필드에 매핑할 수도 있다._
* @InheritConfiguration을 사용하여 매핑 구성 정보를 상속할 수 있다.
* 역방향 매핑 상속의 경우 @InheritInverseConfiguration을 통해 쉽게 반전시킬 수 있다.

**build.gradle**

```
// JDK 11 기준
implementation "org.mapstruct:mapstruct:1.3.0.Final"
annotationProcessor "org.mapstruct:mapstruct-processor:1.3.0.Final"
```

_entity와 dto 코드는 동일하다._\
\
**interface**

```
@Mapper
public interface DataMapper {
        DataMapper INSTANCE = Mappers.getMapper(DataMapper.class);

        @Mapping(source = "entity.sampleInfo", target = "infos")
        SampleDto toDto(SampleEntity entity);

        @Mapping(source = "dto.infos", target = "sampleInfo")
        SampleEntity toEntity(SampleDto dto);
}
```

\
**example**

```
DataMapper dataMapper = DataMapper.INSTANCE;

public void dtoToEntity() {
        SampleDto dto = SampleDto.builder()
                .name("dto")
                .email("...@hello")
                .age(20L)
                .infos(new ArrayList<String>(Collections.singleton("aaa")))
                .build();

        SampleEntity entity = dataMapper.toEntity(dto);
        System.out.println(entity);
    }

public void EntityToDto() {
        SampleEntity entity = SampleEntity.builder()
            .id(1L)
            .name("lob")
            .email("...@test")
            .age(20L)
            .value("value")
            .sampleInfo(new ArrayList<String>(Collections.singleton("aaa")))
            .build();

        SampleDto dto = dataMapper.toDto(entity);
        System.out.println(dto);
}
```

```
SampleEntity(id=null, name=dto, email=...@hello, age=20, sampleInfo=[aaa], value=null)
SampleDto(name=lob, email=...@test, infos=[aaa], age=20)
```

\
**생성된 interface impl**

```
public class MapStructExample$DataMapperImpl implements DataMapper {
    public MapStructExample$DataMapperImpl() {
    }

    public SampleDto toDto(SampleEntity entity) {
        if (entity == null) {
            return null;
        } else {
            SampleDtoBuilder sampleDto = SampleDto.builder();
            List<String> list = entity.getSampleInfo();
            if (list != null) {
                sampleDto.infos(new ArrayList(list));
            }

            sampleDto.name(entity.getName());
            sampleDto.email(entity.getEmail());
            sampleDto.age(entity.getAge());
            return sampleDto.build();
        }
    }

    public SampleEntity toEntity(SampleDto dto) {
        if (dto == null) {
            return null;
        } else {
            SampleEntityBuilder sampleEntity = SampleEntity.builder();
            List<String> list = dto.getInfos();
            if (list != null) {
                sampleEntity.sampleInfo(new ArrayList(list));
            }

            sampleEntity.name(dto.getName());
            sampleEntity.email(dto.getEmail());
            sampleEntity.age(dto.getAge());
            return sampleEntity.build();
        }
    }
}
```

\
**해당 방식의 장, 단점**\
\
**장점**

* 간결한 코드 작성이 가능하다.
* 객체 필드의 변경 사항이 다른 로직에 영향을 주지 않는다.
* 컴파일 시점에 코드를 생성하면서 타입이나 매핑이 불가능한 상태 등의 문제가 발생한 경우 컴파일 에러를 발생시킨다. _이는 상대적으로 런타임에서 안전성을 보장한다._
* 앞서 보았던 자바 코드 매핑 방식과 같은 수준의 성능을 가진다.

**단점**

* 변경 불가능한 필드에 대한 매핑을 제공하지 못한다. (final 필드 - Constructor 주입)
* Lombok Library와 충돌이 발생할 수 있다. (실제로는 Lombok annotation processor가 getter나 builder 등을 만들기 전에 mapstruct annotation processor가 동작하여 매핑할 수 있는 방법을 찾지 못해 발생하는 문제이다. )

&#x20;

&#x20;

&#x20;

#### **ModelMapper**

***

MapStruct와 같이 객체 간의 변환을 위해 사용되는 라이브러리이며, MapStruct와 다른 점은 런타임 시점에 Reflection API를 사용하여 객체를 매핑한다는 것이다.\
\
**build.gradle**

```
implementation group: 'org.modelmapper', name: 'modelmapper', version: '2.3.9'
```

_entity와 dto 코드는 동일하다._\
\
**example**

```
ModelMapper modelMapper = new ModelMapper();

public void dtoToEntity() {
        SampleDto dto = SampleDto.builder()
                .name("dto")
                .email("...@hello")
                .age(20L)
                .infos(new ArrayList<>(Collections.singleton("aaa")))
                .build();

        SampleEntity entity = modelMapper.map(dto, SampleEntity.class);
        System.out.println(entity);
}

public void EntityToDto() {
        SampleEntity entity = SampleEntity.builder()
                .id(1L)
                .name("lob")
                .email("...@test")
                .age(20L)
                .value("value")
                .sampleInfo(new ArrayList<>(Collections.singleton("aaa")))
                .build();

        // 이름이 다른 필드 매핑을 위해 PropertyMap 선언
        PropertyMap<SampleEntity, SampleDto> sampleMap = new PropertyMap<>() {
            @Override
            protected void configure() {
                map().setInfos(source.getSampleInfo());
            }
        };

        // ModelMapper 구성 정보에 PropertyMap 추가
        modelMapper.addMappings(sampleMap);

        // createTypeMap() 을 사용할 수도 있다.

        //modelMapper.createTypeMap(SampleEntity.class, SampleDto.class)
        //        .addMapping(SampleEntity::getSampleInfo, SampleDto::setInfos);

        SampleDto dto = modelMapper.map(entity, SampleDto.class);
        System.out.println(dto);
}

--------
```

```
SampleEntity(id=null, name=dto, email=...@hello, age=20, sampleInfo=null, value=null)
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access using Lookup on org.modelmapper.internal.ProxyFactory (file:/C:/Users/serrl/.gradle/caches/modules-2/files-2.1/org.modelmapper/modelmapper/2.3.9/8bb9110f8df3fbd6c1c2e4b69f7c6add737888e7/modelmapper-2.3.9.jar) to interface java.util.List
WARNING: Please consider reporting this to the maintainers of org.modelmapper.internal.ProxyFactory
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
SampleDto(name=lob, email=...@test, infos=[aaa], age=20)
```

\
**해당 방식의 장, 단점**\
\
**장점**

* 간결한 코드 작성이 가능하다.
* 일반적으로 필드 변경 사항에 대해서 고려하지 않아도 된다.
* Lombok 라이브러리와 충돌없이 같이 사용할 수 있다. (런타임에서 객체를 분석하고 매핑하기 때문에)

**단점**

* Refliection API를 사용하여 객체 필드 정보를 추출하고 매핑하기에 다른 방식보다 상대적으로 성능이 좋지 않다.
  * Reflection API를 사용한다는 이유만으로 느린 것이 아니다.&#x20;
* 바이트코드 생성 방식을 이용하기에 문제 원인 발견과 디버깅이 어렵다. _특정 필드의 변경으로 인한 매핑 누락 문제가 발생하였을 때 이를 인지하기 힘들다._
* 일반적으로 setter를 사용한다. (개인적으로 Setter를 통해 모든 필드를 열어놓는 행위를 싫어한다.)
  *   불변 객체나 Setter를 사용하지 않기 위해 설정하는 방법으로는 별도로 TypeMap과 provider를 정의하고 내부에서 생성자를 이용하는 방법, Converter를 구현하는 방법 (생성자를 로직에 활용하는 것은 Provider와 동일), ModelMapper의 필드 접근 수준을 private로 설정하는 방법이 있다.

      ```
      // 1
      modelMapper.createTypeMap(SampleEntity.class, SampleDto.class)
      		.setProvider( new Provider<SampleDto>() { 
              		public SampleDto get(ProvisionRequest<SampleDto> request) { 
                      		Source source = Source.class.cast(request.getSource()); 
                              return new SampleDto(source.name, source.address); 
                      } 
              });
           
           
      // 2
      Converter<SampleEntity, SampleDto> SampleConverter = () {
          @Override
          public SampleDto convert(MappingContext<SampleEntity, SampleDto> context) {
          	SampleEntity entity = context.getSource();
              return new SampleDto(entity.getName(), entity.getAddress());
          }
      };
              
              
      // 3
      ModelMapper modelMapper = new ModelMapper();
      modelMapper.getConfiguration()
              .setFieldMatchingEnabled(true)
              .setFieldAccessLevel(Configuration.AccessLevel.PRIVATE);
      ```

&#x20;

&#x20;

#### 참고 자료

* [stackabuse.com/guide-to-mapstruct-in-java-advanced-mapping-library/](https://stackabuse.com/guide-to-mapstruct-in-java-advanced-mapping-library/)
* [mapstruct.org/](https://mapstruct.org)

\
\
출처: [https://lob-dev.tistory.com/entry/객체-변환하기-자바-코드-매핑-vs-MapStruct-vs-ModelMapper](https://lob-dev.tistory.com/entry/%EA%B0%9D%EC%B2%B4-%EB%B3%80%ED%99%98%ED%95%98%EA%B8%B0-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%93%9C-%EB%A7%A4%ED%95%91-vs-MapStruct-vs-ModelMapper)&#x20;
