# \[JPA] jpa에서 Repository를 이용한 낙관적락을 구현

### 환경설정

오늘의 테스트를 진행하기 위해서는 Spring과 JPA의 설정이 필요합니다. 해당 설정에 대한 부분은 생략하도록 하겠습니다. 만약 빌드도구로 gradle kotlin dsl 을 이용하신다면 [\[kotlin + Spring\] 코틀린, Spring Boot 환경에서 JPA 사용하기, plugin과 함께](https://sabarada.tistory.com/182)를 참고해주시면 좋을것 같습니다.

### 테스트 Entity

오늘 테스트를 도와줄 Entity는 아래와 같습니다. 고객의 정보를 나타내는 User Entity와 고객이 리뷰를 남긴 Review Entity 입니다.

```
@Entity
@Getter
@DynamicUpdate
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    private String password;

    private String nickname;

    private ThirdPartyType thirdPartyType;

    private Boolean isThirdParty;

    private Integer version;

}
```

### JPA에서 낙관적 락

낙관적락은 버저닝을 통해 락을 구현합니다. JPA에서는 @Version 어노테이션이 붙여서 낙관적락을 구현하는데요. update를 하기전에 버전이 적절한지 체크하도록 합니다. 만약 조회했을 때와 업데이트를 할때 버전이 맞지 않는다면 `OptimisticLockException`를 예외로 throw 하게됩니다. 그리고 업데이트가 성공할경우 version을 올리게됩니다.

그리고 버전은 Integer 뿐만 아니라 Long, Short 등 다양한 Number가 가능합니다. 그리고 Timestamp 자료형 또한 사용할 수 있습니다.

JPA에서 낙관적락의 모드는 아래와같이 4가지를 제공하고 있습니다. 실제로 옵션에 대해서 어떻게 동작하는지는 아래 실습에서 알아보도록 합시다.

* OPTIMISTIC
* OPTIMISTIC\_FORCE\_INCREMENT
* READ - OPTIMISTIC과 동일합니다.
* WRITE - OPTIMISTIC\_FORCE\_INCREMENT과 동일합니다.

### 실습

먼저 Repository 코드를 보도록 하겠습니다. Repository에서 낙관적락은 아래와같이 @Lock 어노테이션을 선언하면 사용할 수 있습니다.

```
public interface UserRepositoryV1 extends JpaRepository<User, Long> {

  @Lock(LockModeType.OPTIMISTIC)
  Optional<User> findWithOptimisticLockById(Long id);
}
```

또한 아래는 Repository를 사용하는 Service의 테스트 코드입니다. 코드의 내용은 User를 가져와서 isThirdParty라는 컬럼을 true로 변경하고 있습니다. 여기서 알아두셔야 할 점은 기본적으로 Lock은 `@Transactional`로 트랜잭션을 잡아주어야 정상적으로 작동한다는 것입니다.

```
@Transactional
public void optimisticLockTest() {
    User user = userRepositoryV1.findWithOptimisticLockById(1L)
            .orElseThrow(() -> new RuntimeException("bad")); // 1번 - 조회

    user.setIsThirdParty(true); // 2번 - 업데이트
}
```

그렇다면 아래에서 코드를 실행했을 때 결과를 하나씩 보면서 사용하는 방법을 알아보도록 합시다.

#### OPTIMISTIC 옵션으로 이용

가장먼저 서비스 로직에서 2번 업데이트 코드는 제거하고 1번 조회만 해보도록 하겠습니다. 아래와 같이 2가지의 쿼리가 노출되게 됩니다. 가장먼저 노출되는 것은 user 정보를 조회를 하는 쿼리입니다.

```
select
    user0_.id as id1_12_,
    user0_.created_datetime as created_2_12_,
    user0_.updated_datetime as updated_3_12_,
    user0_.email as email4_12_,
    user0_.is_third_party as is_third5_12_,
    user0_.nickname as nickname6_12_,
    user0_.password as password7_12_,
    user0_.third_party_type as third_pa8_12_,
    user0_.version as version9_12_ 
from
    user user0_ 
where
    user0_.id=?
```

그리고 Transaction 커밋 바로전에 select 쿼리가 한번더 나오는 것을 확인할 수 있었습니다.

```
select
    version 
from
    user 
where
    id =?
```

2번의 업데이트 코드를 추가하고 실행해보도록 하겠습니다. 그랬을 경우 가장 먼저 user 정보를 조회합니다. 그리고 업데이트 쿼리가 나타납니다. 그리고 마지막에 다시 Transaction 커밋전에 select 쿼리가 한번 더 나옵니다. 결과적으로 version이 이경우 업데이트되게 됩니다.

```
update
    user 
set
    is_third_party=?,
    version=? 
where
    id=? 
    and version=?
```

마지막으로 트랜잭션 도중에 다른 사람이 먼저 업데이트를 진행하여 버전을 변경했을 경우에 발생하는 에러는 `ObjectOptimisticLockingFailureException`과 `StaleStateException`입니다. 그리고 detail 메시지로는 `Batch update returned unexpected row count from update [0]; actual row count: 0; expected: 1`가 나오는 것을 확인할 수 있었습니다.

#### OPTIMISTIC\_FORCE\_INCREMENT

Repository의 `@Lock` 어노테이션의 `LockModeType`를 `OPTIMISTIC_FORCE_INCREMENT`으로 변경한 후 진행해보도록 하겠습니다. 가장먼저 2번 업데이트를 없애고 조회만 진행했을 때입니다. 그랬을 경우 아래와 같이 정보조회 쿼리가 먼저 발생합니다.

```
select
    user0_.id as id1_12_,
    user0_.created_datetime as created_2_12_,
    user0_.updated_datetime as updated_3_12_,
    user0_.email as email4_12_,
    user0_.is_third_party as is_third5_12_,
    user0_.nickname as nickname6_12_,
    user0_.password as password7_12_,
    user0_.third_party_type as third_pa8_12_,
    user0_.version as version9_12_ 
from
    user user0_ 
where
    user0_.id=?
```

그리고 특이하게 Transaction 커밋 바로전에 update 쿼리가 발생합니다. 결과적으로 version이 1 증가되게 됩니다.

```
update
    user 
set
    version=? 
where
    id=? 
    and version=?
```

2번 코드를 살려 해당 row를 업데이트를 하게되면 추가적으로 해당 업데이트를 반영하는 쿼리가 추가적으로 실행됩니다.

```
update
    user 
set
    is_third_party=?,
    version=? 
where
    id=? 
    and version=?
```

결과적으로 2번의 업데이트가 발생합니다. 그리고 version은 1이 아닌 2가 올라가게됩니다.

### 마무리

오늘은 이렇게 JPA에서 낙관적락을 사용하는 방법과 그랬을 때 나오는 쿼리를 알아보는 시간을 가져보았습니다.

다음시간에는 비관적락을 사용하는 방법과 그랬을 때 나오는 쿼리에 대해서 알아보는 시간을 가져보도록 하겠습니다.

감사합니다.

### 참조

[baeldung\_jpa-optimistic-locking](https://www.baeldung.com/jpa-optimistic-locking)
