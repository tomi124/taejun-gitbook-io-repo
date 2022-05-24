# JPA 정리

## 인프런 김영한님의 '실전! 스프링 데이터 JPA' 를 학습 후 핵심을 정리한 내용 <a href="#jpa" id="jpa"></a>

> 인프런 김영한님의 '실전! 스프링 데이터 JPA' 강의 보러가기\
> [https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84#](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84)

### 공통 인터페이스 적용 <a href="#undefined" id="undefined"></a>

* 인터페이스를 생성 후 JpaRepository\<T, ID> 인터페이스를 상속 받는다.
* T: 타겟 엔티티
* ID: 타겟 엔티티의 PK 자료형

```
public interface MemberRepository extends JpaRepository<Member, Long> {
    ...
}
```

### 메소드 이름으로 쿼리 생성 <a href="#undefined" id="undefined"></a>

* 스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행
* 조회: find…By ,read…By ,query…By get…By,

```
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

> 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.\
> 그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.\
> 이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 매우 큰 장점이다.

#### @Query, 리포지토리 메소드에 쿼리 정의하기 <a href="#query" id="query"></a>

* @org.springframework.data.jpa.repository.Query 어노테이션을 사용
* 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리라 할 수 있음
* JPA Named 쿼리처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있음(매우 큰 장점!)

```
public interface MemberRepository extends JpaRepository<Member, Long> {
@Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```

> 참고: 실무에서는 메소드 이름으로 쿼리 생성 기능은 파라미터가 증가하면 메서드 이름이 매우\
> 지저분해진다. 따라서 @Query 기능을 자주 사용하게 된다.

#### DTO 조회하기 <a href="#dto" id="dto"></a>

```
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
List<MemberDto> findMemberDto();
```

> 주의! DTO로 직접 조회 하려면 JPA의 new 명령어를 사용해야 한다. 그리고 다음과 같이 생성자가 맞는 DTO 가 필요하다. (JPA와 사용방식이 동일하다.)

#### 파라미터 바인딩 <a href="#undefined" id="undefined"></a>

```
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username = :name")
    Member findMembers(@Param("name") String username);
}
```

#### 스프링 데이터 JPA 페이징과 정렬 <a href="#jpa" id="jpa"></a>

```
public interface MemberRepository extends JpaRepository<Member, Long> {
    Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
    Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안함
    List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안함
    List<Member> findByUsername(String name, Sort sort);
}
```

* 두 번째 파라미터로 받은 **Pagable** 은 인터페이스다. 따라서 실제 사용할 때는 해당 인터페이스를 구현한\
  org.springframework.data.domain.PageRequest 객체를 사용한다.
* PageRequest 생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를\
  입력한다. 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 참고로 페이지는 0부터 시작한다.\


**PageRequest 를 통해 파라미터 바인딩**

```
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
Page<Member> page = memberRepository.findByAge(10, pageRequest);
```

**count 쿼리를 다음과 같이 분리할 수 있음**

```
@Query(value = “select m from Member m”,
countQuery = “select count(m.username) from Member m”)
Page<Member> findMemberAllCountBy(Pageable pageable);
```

### 벌크성 수정 쿼리 <a href="#undefined" id="undefined"></a>

```
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

* 벌크성 수정, 삭제 쿼리는 @Modifying 어노테이션을 사용
* 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화: @Modifying(clearAutomatically = true)
* 이 옵션 없이 회원을 findById 로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아서 문제가 될 수\
  있다. 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자

### @EntityGraph <a href="#entitygraph" id="entitygraph"></a>

* 연관된 엔티티들을 SQL 한번에 조회하는 방법
* member team은 지연로딩 관계이다. 따라서 다음과 같이 team의 데이터를 조회할 때 마다 쿼리가\
  실행된다. (N+1 문제 발생)

```
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(String username)
```

**EntityGraph 정리**

* 사실상 페치 조인(FETCH JOIN)의 간편 버전
* LEFT OUTER JOIN 사용

### JPA Hint <a href="#jpa-hint" id="jpa-hint"></a>

```
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);
```

* readOnly 로 적용하였기 때문에, 해당 쿼리로 조회된 엔티티 객체는 스냅샷을 만들지 않아 변경 감지가 일어나지 않아 변경되지 않는다.

### Web 확장 - 도메인 클래스 컨버터 <a href="#web" id="web"></a>

```
@RestController
@RequiredArgsConstructor
public class MemberController {

private final MemberRepository memberRepository;

  @GetMapping("/members/{id}")
  public String findMember(@PathVariable("id") Member member) {
    return member.getUsername();
  }
}
```

*   HTTP 요청은 회원 id 를 받지만 도메인 클래스 컨버터가 중간에 동작해서 회원 엔티티 객체를 반환

    > 주의: 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, 이 엔티티는 단순 조회용으로만 사용해야 한다.\
    > (트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.)

### Web 확장 - 페이징과 정렬 <a href="#web" id="web"></a>

```
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
  Page<Member> page = memberRepository.findAll(pageable);
  return page;
}
```

* 요청 예)\
  \
  /members?page=0\&size=3\&sort=id,desc\&sort=username,desc
* page: 현재 페이지, 0부터 시작한다.
* size: 한 페이지에 노출할 데이터 건수
* sort: 정렬 조건을 정의한다. 예) 정렬 속성,정렬 속성...(ASC | DESC), 정렬 방향을 변경하고 싶으면 sort\
  파라미터 추가 ( asc 생략 가능)

#### 글로벌 설정 <a href="#undefined" id="undefined"></a>

```
spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/
spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
```

#### 개별 설정 <a href="#undefined" id="undefined"></a>

* @PageableDefault 어노테이션을 사용

```
@GetMapping(value = "/members_page", method = RequestMethod.GET)
public String list(@PageableDefault(size = 12, sort = “username”, direction = Sort.Direction.DESC) Pageable pageable) {
  ...
}
```

#### Page 를 1부터 시작하기 <a href="#page-1" id="page-1"></a>

> Pageable, Page를 파리미터와 응답 값으로 사용히지 않고, 직접 클래스를 만들어서 처리한다. 그리고\
> 직접 PageRequest(Pageable 구현체)를 생성해서 리포지토리에 넘긴다. 물론 응답값도 Page 대신에\
> 직접 만들어서 제공해야 한다

### 스프링 데이터 JPA 분석 <a href="#jpa" id="jpa"></a>

```
SimpleJpaRepository.class

@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> ...{

    @Transactional
    public <S extends T> S save(S entity) {
      if (entityInformation.isNew(entity)) {
      em.persist(entity);
      return entity;
      } else {
      return em.merge(entity);
      }
  }
...
}
```

* @Repository 적용: JPA 예외를 스프링이 추상화한 예외로 변환
* @Transactional 트랜잭션 적용\

  1. JPA의 모든 변경은 트랜잭션 안에서 동작
  2. 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드를 트랜잭션 처리
  3. 서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션 시작
  4. 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 전파 받아서 사용
  5. 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능했음(사실은 트랜잭션이 리포지토리 계층에 걸려있는 것임)
* @Transactional(readOnly = true)
  1. 데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 readOnly = true 옵션을 사용하면\
     플러시를 생략해서 약간의 성능 향상을 얻을 수 있음\


### 새로운 엔티티를 구별하는 방법 <a href="#undefined" id="undefined"></a>

#### save() 메서드 <a href="#save" id="save"></a>

* 새로운 엔티티면 저장( persist )
* 새로운 엔티티가 아니면 병합( merge )

#### 새로운 엔티티를 판단하는 기본 전략 <a href="#undefined" id="undefined"></a>

* 식별자가 객체일 때 null 로 판단
* 식별자가 자바 기본 타입일 때 0 으로 판단
*   Persistable 인터페이스를 구현해서 판단 로직 변경 가능

    > JPA 식별자 생성 전략이 @GenerateValue 면 save() 호출 시점에 식별자가 없으므로 새로운\
    > 엔티티로 인식해서 정상 동작한다. 그런데 JPA 식별자 생성 전략이 @Id 만 사용해서 직접 할당이면 이미\
    > 식별자 값이 있는 상태로 save() 를 호출한다. 따라서 이 경우 merge() 가 호출된다. merge() 는 우선\
    > DB를 호출해서 값을 확인하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율 적이다. 따라서\
    > Persistable 를 사용해서 새로운 엔티티 확인 여부를 직접 구현하게는 효과적이다.\
    > 참고로 등록시간( @CreatedDate )을 조합해서 사용하면 이 필드로 새로운 엔티티 여부를 편리하게 확인할\
    > 수 있다. (@CreatedDate에 값이 없으면 새로운 엔티티로 판단)

\
