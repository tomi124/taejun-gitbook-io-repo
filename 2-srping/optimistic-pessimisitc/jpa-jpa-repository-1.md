# \[JPA] jpa에서 Repository를 이용한 비관적락을 구현

### 비관적 락

비관적락은 내가 접근하고 하는 Database 리소스에 다른사람이 접근조차 하지못하도록 락을 걸고 작업을 진행하는 것을 말합니다. 물론 여기서 접근이라는 것은 READ 작업과 WRITE 작업이 분할되어 있습니다. 경우에 맞춰 둘다 불가능할지 아니면 하나만 가능할지를 정하는것이 가능합니다. 비관적락을 사용할 때 2가지 옵션을 선택할 수 있습니다. 배타락(exclusive lock)과 공유락(shared lock)입니다. 공유락을 걸면 다른 트랜잭션에서는 읽기는 가능하지만 쓰기는 불가능힙니다. 베타락에서는 다른 트랜잭션에서 읽기와 쓰기가 모두 불가능합니다. 배타락은 쿼리로 보면 `SELECT … FOR UPDATE`로 나타낼 수 있습니다.

JPA에서 비관적 락의 옵션을 3가지입니다. 아래 옵션들은 낙관적락과 마찬가지로 `LockModeType` enum으로 설정할 수 있습니다. 아래의 락 설정들은 해당 트랜잭션이 commit 되거나 rollback 될때까지 유지됩니다.

* PESSIMISTIC\_READ – 해당 리소스에 공유락을 겁니다. 타 트랜잭션에서 읽기는 가능하지만 쓰기는 불가능해집니다.
* PESSIMISTIC\_WRITE – 해당 리소스에 베타락을 겁니다. 타 트랜잭션에서는 읽기와 쓰기 모두 불가능해집니다. (DBMS 종류에 따라 상황이 달라질 수 있습니다)
* PESSIMISTIC\_FORCE\_INCREMENT – 해당 리소스에 베타락을 겁니다. 타 트랜잭션에서는 읽기와 쓰기 모두 불가능해집니다. 추가적으로 낙관적락처럼 버저닝을 합니다. 따라서 버전에 대한 컬럼이 필요합니다.

비관적 락에서 발생할 수 있는 예외상황은 아래와 같습니다.

* PessimisticLockException – 트랜잭션에서 락 취득을 하지 못해서 생기는 예외
* LockTimeoutException – 트랜잭션에서 락 취득을 타임아웃 시간내에 해지 못해서 생기는 예외
* PersistanceException – 값을 변경할 때 생길 수 있는 예외상황 (NoResultException, NoUniqueResultException, LockTimeoutException, QueryTimeoutException) 등등 이럴경우 결과적으로 롤백됩니다.

이 외에도 낙관적락에는 없는 락 범위 설정 등이 있습니다. 이부분은 실습을 통해서 알아보도록 하겠습니다.

### 실습

실습을 통해서 비관적락을 JPA에서 사용하는 방법과 도출되는 쿼리 그리고 사용할 수 있는 옵션에 대해서 알아보도록 하겠습니다. 설정 및 Entity, Repository 그리고 Service는 이전 [\[JPA\] jpa에서 Repository를 이용한 낙관적락을 구현해봅시다.](https://sabarada.tistory.com/185)과 동일합니다. 추가적으로 DB는 MariaDB를 사용하고 있습니다.

#### 테스트 Entity

비관적 락에서 사용할 Entity 입니다. 낙관적 락의 테스트와는 달리 이번에는 버저닝에 대한 부분인 `version` 컬럼은 삭제하였습니다.

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
}
```

#### 테스트 Repository 및 Service

Repository 및 Service는 크게 변경된 점이 없습니다.

```
public interface UserRepositoryV1 extends JpaRepository<User, Long>, UserRepositoryV1Custom {

  @Lock(LockModeType.PESSIMISTIC_READ) // LockModeType으로 Option 변경 가능
  Optional<User> findWithPessimisticLockById(Long id);
}
```

```
@Transactional
public void pessimisticLockTest() {
    User user = userRepositoryV1.findWithPessimisticLockById(1L) // 1번 - 조회
            .orElseThrow(() -> new RuntimeException("bad"));

    user.setIsThirdParty(true); // 2번 - 업데이트 
}
```

#### PESSIMISTIC\_READ

`LockModeType.PESSIMISTIC_READ` 옵션으로 진행했을 때 발생하는 쿼리들을 확인해보도록 하겠습니다. 먼저 아래는 성공적으로 update 될때 출력되는 쿼리입니다. select를 하며 `in share mode`가 출력되며 update 쿼리도 정상출력되어 결과적으로 업데이트가 완료됨을 알 수 있었습니다.

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
    user0_.id=? lock in share mode

update
    user 
set
    is_third_party=? 
where
    id=?
```

2곳에서 동시에 발생시켜 LockException이 발생했을때는 아래와 같은 stacktrace가 잡혔습니다. 에러 로그 메시지로는 Dealock 을 찾았다는 말과 함께 예외가 아래처럼 발생하는 것을 알 수 있었습니다.

```
2021-08-15 18:04:45.802 ERROR 65445 --- [       Thread-9] o.h.engine.jdbc.spi.SqlExceptionHelper   : (conn=277) Deadlock found when trying to get lock; try restarting transaction
Exception in thread "Thread-9" org.springframework.dao.CannotAcquireLockException: could not execute batch; SQL
Caused by: org.hibernate.exception.LockAcquisitionException: could not execute batch
[...중략...]
Caused by: java.sql.BatchUpdateException: (conn=277) Deadlock found when trying to get lock; try restarting transaction
[...중략...]
Caused by: java.sql.SQLTransactionRollbackException: (conn=277) Deadlock found when trying to get lock; try restarting transaction
[...중략...]
Caused by: org.mariadb.jdbc.internal.util.exceptions.MariaDbSqlException: Deadlock found when trying to get lock; try restarting transaction
[...중략...]
Caused by: java.sql.SQLException: Deadlock found when trying to get lock; try restarting transaction
```

#### PESSIMISTIC\_WRITE

`LockModeType.PESSIMISTIC_WRITE` 옵션을 사용하여 쿼리를 실행했을 때에는 아래와 같이 select 쿼리의 마지막에 `for update`가 붙는것을 알 수 있었습니다.

```
select
    user0_.id as id1_12_,
    user0_.created_datetime as created_2_12_,
    user0_.updated_datetime as updated_3_12_,
    user0_.email as email4_12_,
    user0_.is_third_party as is_third5_12_,
    user0_.nickname as nickname6_12_,
    user0_.password as password7_12_,
    user0_.third_party_type as third_pa8_12_ 
from
    user user0_ 
where
    user0_.id=? for update

update
    user 
set
    is_third_party=? 
where
    id=?
```

그렇다면 2개의 트랜잭션을 동시에 하나의 리소스에 접근시키면 어떻게 될까요 ? 처음에 저는 예외를 에상했지만 예외는 나지 않았습니다. 결과적으로 for update가 걸려 있을때 MariaDB에서는 하나의 리소스에 대해서 여러 READ의 경우 Error가 나진 않고 `MVCC`를 통해서 `select ... for update`는 동시에 가능한것을 확인하였습니다. 그리고 `for update`가 붙어있을 때 업데이트가 진행되면 Lock 이걸려 있는 순서대로 update가 진행되는 점을 확인할 수 있었습니다. `MVCC`를 지원하지 않는 DB를 사용한다면 멀티 READ시 에러가 날 것입니다.

#### 락 범위 (Lock Scope)

비관적락은 락 범위(Lock Scope)를 지정할수 있습니다. 락 범위라는 것은 다른 테이블과 Join을 해서 사용할 때 lock을 메인이 되는 1개의 테이블에만 걸 것인지 아니면 참여하는 모든 테이블의 row에 Lock을 걸 것인지 지정하는 것입니다. 해당 옵션은 `PessimisticLockScope`을 통해서 걸 수 있습니다. Repository에 거는 방법은 `@Queryhints` 및 `@Queryhint`를 이용하시면 됩니다.

```
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({@QueryHint(name = "javax.persistence.lock.scope", value = "EXTENDED")})
Optional<Review> findWithPessimisticLockById(Long id);
```

아쉽게도 모든 DBMS에서 제공해주고 있지 않습니다. MariaDB에서는 동작하지 않는것을 확인하였습니다.

#### Lock Timeout 설정

Lock Timeout은 락을 잡고 있는 최대 시간을 지정하는 것입니다. `LockModeType.PESSIMISTIC_READ`으로 진행했을 때 Lock Timeout이 걸려 있다면 Lock이 걸려있다고 바로 에러를 내지 않습니다. 해당 시간은 대기하고 예외처리를 하는것을 확인할 수 있었습니다. 사용방법은 아래와 같습니다.

```
public interface UserRepositoryV1 extends JpaRepository<User, Long>, UserRepositoryV1Custom {

  @Lock(LockModeType.PESSIMISTIC_READ)
  @QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value ="10000")})
  Optional<User> findWithPessimisticLockById(Long id);
}
```

MariaDB를 기준으로 `LockModeType.PESSIMISTIC_READ`, `LockModeType.PESSIMISTIC_WRITE`를 테스트했을 때 아래와 같이 wait이 걸리는 걸 확인할 수 있었습니다. 아래는 `LockModeType.PESSIMISTIC_WRITE`의 결과입니다.

```
select
    user0_.id as id1_12_,
    user0_.created_datetime as created_2_12_,
    user0_.updated_datetime as updated_3_12_,
    user0_.email as email4_12_,
    user0_.is_third_party as is_third5_12_,
    user0_.nickname as nickname6_12_,
    user0_.password as password7_12_,
    user0_.third_party_type as third_pa8_12_ 
from
    user user0_ 
where
    user0_.id=? for update
        wait 1000
```

###

### 참조

[stackoverflow\_how-to-specify-lock-timeout-in-spring-data-jpa-query](https://stackoverflow.com/questions/29765934/how-to-specify-lock-timeout-in-spring-data-jpa-query)

[baeldung\_jpa-pessimistic-locking](https://www.baeldung.com/jpa-pessimistic-locking)
