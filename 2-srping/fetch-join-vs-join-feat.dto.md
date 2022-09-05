# Fetch Join vs 일반 Join(feat.DTO)



> 💡 Spring Data JPA를 사용하다보면 연관관계를 갖고 있는 두 엔티티에 대해 조회를 할 때 N+1 문제가 발생합니다.\
> 이전 프로젝트를 진행하면서 N+1 문제가 발생할 수 있는 상황에서 직접 @Query에 Join query를 작성하면서 N+1 문제를 직접적으로 확인하고 체감하진 못하였는데요,\
> 나중에 N+1 문제의 해결방법을 찾아보았을 때, @EntityGraph 를 사용하는 방법과 fetch join을 사용하는 방법등이 있었습니다.\
> 그래서 단순히 이거 그냥 Join Query 사용하면 되는거 아니야? 라고 생각하고 넘어갔었습니다 🤭 하지만 일반 Join과 Fetch Join 간의 차이점을 모르고 사용했다는게 계속 마음이 걸렸습니다. 🥲 \
> 그래서 **테스트**를 통해 둘 간의 차이점을 알아보고자 해당 포스팅을 준비하게 되었습니다. 🔥

#### 테스트 환경 세팅 <a href="#undefined" id="undefined"></a>

***

**User entity**

***

> 💡 사용자(User)는 name을 갖는다.

```java
public class User {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  private Long id;

  @Column(name = "name")
  private String name;

}
```

**Post entity**

***

> 💡 게시물(Post)는 title, description을 갖고, 사용자(User) 한명이 작성할 수 있다.

```java
public class Post {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  private Long id;

  @ManyToOne(fetch = FetchType.EAGER)
  @JoinColumn(name = "user_id", referencedColumnName = "id")
  private User user;

  @Column
  private String title;

  @Column
  private String description;

}
```

**@BeforeEach setUp Method**

***

> 💡 사용자(user) - 게시물(post) 관계를 5개 생성한다.

```java
@BeforeEach
voidsetUp() {
  userRepository.deleteAll();
  postRepository.deleteAll();

  System.out.println("==== setUp start ====");

for(int i = 0; i < 5; i++) {
    User user = User.builder()
        .name("user" + i)
        .build();

    userRepository.save(user);

    Post post = Post.builder()
        .title("title" + i)
        .description("description" + i)
        .user(user)
        .build();

    postRepository.save(post);
  }
  System.out.println("==== setUp end ====");

}
```

### 1. 테스트 <a href="#1" id="1"></a>

***

> 💡 테스트 요구사항은 다음과 같습니다.\
> **"모든 게시물을 조회하고, 조회한 게시물의 작성자 이름을 출력한다."**

#### A. N+1 문제를 확인한다. <a href="#a-n1" id="a-n1"></a>

***

```java
@Test
@DisplayName("N+1문제를 확인한다.")
void NplusOneTest() {
  List<Post> posts = postRepository.findAll();

  posts.stream()
      .map(post -> post.getUser().getName())
      .forEach(System.out::println);
}
```

**실행 결과**\
![](https://velog.velcdn.com/images%2Fheoseungyeon%2Fpost%2F141f0442-2d8d-46a7-b511-cca48b515cd7%2Fimage.png)

조회하고자 하는 엔티티(Post)의 갯수(5)만큼 매핑된 엔티티(User)에 대해 조회 query를 날립니다.

→ N(User 조회) + 1(Post 조회)

→ 1번 조회할려고 하는데 N번 조회를 더하게 되네.

**→ 이게 바로 N+1 문제**

일반적으로 SQL을 공부하였다면 연관 관계가 있는 테이블의 데이터를 조회할 때 Join 쿼리를 사용하여 한 번에 조회를 하면 되겠네 라는 생각을 할 수 있습니다.

그러면 **B(Fetch Join), C(일반 Join) 테스트**를 통해 N+1 문제를 해결해보도록 하겠습니다.

#### B. Fetch Join 쿼리 사용하여 N+1 문제를 해결한다. (성공) <a href="#b-fetch-join-n1" id="b-fetch-join-n1"></a>

***

Fetch **조인으로 Post 데이터를 받아오는 쿼리**

```java
@Query("SELECT p FROM Post p "
      + "JOIN FETCH p.user")
List<Post> findAllByFetchJoin();
```

**테스트 코드**

```java
@Test
  @DisplayName("Fetch Join 쿼리 사용하여 N+1 문제를 해결한다.")
  void fetchJoinTest() {
    List<Post> posts = postRepository.findAllByFetchJoin();

    posts.stream()
        .map(post -> post.getUser().getName())
        .forEach(System.out::println);
  }
```

**실행 결과**

```java
Hibernate: 
select post0_.id as id1_0_0_, user1_.id as id1_1_1_, post0_.description as descript2_0_0_, post0_.title as title3_0_0_, post0_.user_id as user_id4_0_0_, user1_.name as name2_1_1_ 
from post post0_ 
inner join user user1_ 
on post0_.user_id=user1_.id

// 출력 결과 
user0
user1
user2
user3
user4
```

_**결과 분석**_

fetch join을 사용하니 게시물을 조회하는 쿼리에서 단일 게시물이 갖고 있는 작성자(User)의 모든 데이터까지 **하나의 쿼리문**으로 조회하는 것을 확인할 수 있습니다.

#### C. 일반 Join 쿼리 사용하여 N+1 문제를 해결한다. (실패) <a href="#c-join-n1" id="c-join-n1"></a>

***

**일반 조인으로 Post 데이터를 받아오는 쿼리**

```java
@Query("SELECT p FROM Post p "
      + "LEFT JOIN User u "
      + "ON p.user.id = u.id")
List<Post> findAllByJoin();
```

**테스트 코드**

```java
@Test
@DisplayName("일반 Join 쿼리 사용하여 N+1 문제를 해결한다.")
void joinTest() {
  List<Post> posts = postRepository.findAllByJoin();

  posts.stream()
      .map(post -> post.getUser().getName())
      .forEach(System.out::println);
}
```

**실행 결과**

```java
Hibernate: select post0_.id as id1_0_, post0_.description as descript2_0_, post0_.title as title3_0_, post0_.user_id as user_id4_0_ from post post0_ left outer join user user1_ on (post0_.user_id=user1_.id)
Hibernate: select user0_.id as id1_1_0_, user0_.name as name2_1_0_ from user user0_ where user0_.id=?
Hibernate: select user0_.id as id1_1_0_, user0_.name as name2_1_0_ from user user0_ where user0_.id=?
Hibernate: select user0_.id as id1_1_0_, user0_.name as name2_1_0_ from user user0_ where user0_.id=?
Hibernate: select user0_.id as id1_1_0_, user0_.name as name2_1_0_ from user user0_ where user0_.id=?
Hibernate: select user0_.id as id1_1_0_, user0_.name as name2_1_0_ from user user0_ where user0_.id=?

// 출력 결과 
user0
user1
user2
user3
user4
```

_**결과 분석**_

일반 join을 사용하니 게시물을 조회하는 쿼리에서 단일 게시물이 갖고 있는 작성자(User)의 id(PK)값만 조회하는 것을 확인할 수 있습니다.

그 후 조회한 user\_id를 통해 N번의 User 조회 쿼리를 날리는 것을 확인할 수 있습니다.

_**어라?!**_

**분명** 이전 프로젝트에서 일반 join을 사용하더라도 N+1 문제를 확인했는데, 테스트를 진행해보니 N+1 문제가 발생하는 것을 확인할 수 있습니다. 🤦🏻

슉 슈슈슉 슉슉! (프로젝트 코드 확인하러 가는중,,,) 💨

이전 프로젝트의 일반 join 쿼리를 확인해보니 두 엔티티의 데이터를 가져올 때 DTO 객체를 반환값으로 설정한 것을 확인할 수 있었습니다.

그러면 **D(일반 Join with DTO) 테스트**를 통해 N+1 문제가 해결되었는지 확인해보겠습니다.

#### D. 일반 join을 사용하고 DTO 를 만들어 N+1 문제를 해결한다 (성공) <a href="#d-join-dto-n1" id="d-join-dto-n1"></a>

***

**PostUserDTO : user 이름과 post 제목을 갖는다.**

```java
public class PostUserDTO {

  private String userName;

  private String postTitle;

}
```

**일반 조인으로 DTO 객체로 데이터를 받아오는 쿼리**

```java
@Query("SELECT new com.example.springbootstudy.domain.PostUserDTO(u.name, p.title) FROM Post p "
      + "LEFT JOIN User u "
      + "ON p.user.id = u.id")
List<PostUserDTO> findAllPostUserByFetchJoin();
```

**테스트 코드**

```java
@Test
@DisplayName("일반 Join 쿼리 와 DTO 를 사용하여 N+1 문제를 해결한다.")
void joinWithDTOTest() {
  List<PostUserDTO> postUserDTOS = postRepository.findAllPostUserByFetchJoin();

  postUserDTOS.stream()
      .map(postUser -> postUser.getUserName())
      .forEach(System.out::println);
}
```

**실행 결과**

```java
Hibernate: select user1_.name as col_0_0_, post0_.title as col_1_0_ from post post0_ left outer join user user1_ on (post0_.user_id=user1_.id)

// 출력 결과 
user0
user1
user2
user3
user4
```

_**결과 분석**_

JPQL에선 조회 쿼리문에서 DTO를 반환하는 기능을 제공합니다.

해당 기능을 활용하여 PostUserDTO 를 통해 조회 결과를 반환받으니 N+1 문제가 발생하지 않는 것을 확인할 수 있었습니다.

### 2. Fetch Join 과 일반 Join <a href="#2-fetch-join-join" id="2-fetch-join-join"></a>

***

위 테스트를 통해 Fetch Join과 일반 Join을 결과로써 차이점을 확인해볼 수 있었는데요,

우리가 확인했던 결과는 다음과 같습니다.

**Fetch Join**

* 게시물을 조회하는 쿼리에서 단일 게시물이 갖고 있는 작성자(User)의 모든 데이터를 하나의 쿼리문으로 조회

**일반 Join**

* 게시물을 조회하는 쿼리에서 단일 게시물이 갖고 있는 작성자(User)의 id(PK)값만 조회
* 그 후, 획득한 user\_id를 통해 User 조회 쿼리 N번 수행 (FetchType.EAGER)

_그러면 Fetch Join과 일반 Join의 차이점은 무엇일까요?_

**Fetch Join**

* 조회 주체가 되는 엔티티와 연관 관계의 엔티티(JOIN) 까지 모두 조회하여 영속화한다.
* 즉, 2개의 엔티티 모두 영속성 컨텍스트로 관리되어진다.

**일반 Join**

* 조회 주체가 되는 엔티티만 조회하고 영속화한다.
* 만약 연관 관계의 엔티티 데이터를 사용해야 할 경우 별도의 조회 쿼리문을 실행 해야 함.
  * FetchType.EAGER 일 경우, 연관 관계의 엔티티를 영속화하기 위해 N번의 쿼리를 발생시킴.
  * FetchType.LAZY 일 경우, 최초 조회시 획득한 id 로 조회를 N번해야함.

### 3. 일반 Join with DTO <a href="#3-join-with-dto" id="3-join-with-dto"></a>

***

Fetch Join을 사용하여 N+1문제를 해결하는 것이 나이스한 방법인 것은 알게된 것 같습니다.

하지만 위 테스트에서 확인할 수 있듯이 주어진 요구사항이 단순히 데이터 조회만을 수행하는 것이면 **D(일반 Join with DTO)** 와 같이 코드를 작성하더라도 N+1 문제 걱정 없이 요구사항을 해결할 수 있습니다.

### 4. Fetch Join vs 일반 Join with DTO <a href="#4-fetch-join-vs-join-with-dto" id="4-fetch-join-vs-join-with-dto"></a>

***

그렇다면 Fetch Join 과 일반 Join with DTO 방법은 각각 언제 사용해야 좋은 것일까요?

**Fetch Join**

***

* 두 엔티티의 영속화가 필요로 할 때

**Join with DTO**

***

* 영속화 없이 데이터 조회만 할 수 있을 경우
* 두 엔티티의 컬럼이 무수히 많은 경우(필요한 컬럼만 조회할 수 있음)
* 검색조건으로 연관관계 엔티티를 활용할 경우
