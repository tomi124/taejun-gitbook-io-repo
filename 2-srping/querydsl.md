# queryDsl



### QueryDSL이란? <a href="#querydsl" id="querydsl"></a>

> Querydsl 정적 타입을 이용해서 SQL과 같은 쿼리를 생성할 수 있도록 해 주는 프레임워크다.
>
> [Querydsl - 레퍼런스 문서](http://www.querydsl.com/static/querydsl/3.4.3/reference/ko-KR/html\_single/)

### QueryDSL 왜 사용할까? <a href="#querydsl" id="querydsl"></a>

> Querydsl은 타입에 안전한 방식으로 HQL 쿼리를 실행하기 위한 목적으로 만들어졌다. HQL 쿼리를 작성하다보면 String 연결을 이용하게 되고, 이는 결과적으로 읽기 어려운 코드를 만드는 문제를 야기한다. String을 이용해서 도메인 타입과 프로퍼티를 참조하다보면 오타 등으로 잘못된 참조를 하게 될 수 있으며, 이는 String을 이용해서 HQL 작성할 때 발생하는 또 다른 문제다.
>
> 타입에 안전하도록 도메인 모델을 변경하면 소프트웨어 개발에서 큰 이득을 얻게 된다. 도메인의 변경이 직접적으로 쿼리에 반영되고, 쿼리 작성 과정에서 코드 자동완성 기능을 사용함으로써 쿼리를 더 빠르고 안전하게 만들 수 있게 된다
>
> [Querydsl - 레퍼런스 문서](http://www.querydsl.com/static/querydsl/3.4.3/reference/ko-KR/html\_single/)

### QueryDSL 사용 <a href="#querydsl" id="querydsl"></a>

#### gradle 설정 <a href="#gradle" id="gradle"></a>

```java
plugins {
    // ...
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10" // 추가
    // ...
}

// ...

dependencies {
    // ...
    implementation 'com.querydsl:querydsl-jpa' // 추가
    // ...
}

// ...

// queryDSL이 생성하는 QClass 경로 설정
def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```

#### Q클래스 만들기 <a href="#q" id="q"></a>

gradle 설정이 끝났으면 Q클래스를 만들어보자.

![](https://media.vlpt.us/images/tigger/post/3e5a5eb6-de64-4387-b045-5af37e427a86/image.png)

만드는 방법은 위의 그림과 같이 먼저 Gradle Project(View → Tool Windows → Gradle Project)를 열고 Tasks → other → compileJava를 실행시키면 build → generated에 Q클래스가 생성된다.

![](https://media.vlpt.us/images/tigger/post/62828f3e-c242-45b9-8273-16d3773abca1/image.png)

#### Config 설정 <a href="#config" id="config"></a>

```java
@Configuration
public class QueryDSLConfig {

    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```

이제 JPAQueryFactory를 주입받아 QueryDSL을 사용할 수 있다.

#### 사용법 <a href="#undefined" id="undefined"></a>

**1. 기본 사용법**

**Post 엔티티**

```java
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String content;

    // ...
}
```

**PostRepository**

```java
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

**PostRepositorySupport**

```java
@Repository
public class PostRepositorySupport extends QuerydslRepositorySupport {

    private final JPAQueryFactory jpaQueryFactory;

    public PostRepositorySupport(final JPAQueryFactory jpaQueryFactory) {
        super(Post.class);
        this.jpaQueryFactory = jpaQueryFactory;
    }

    public List<Post> findByTitle(final String title) {
        return jpaQueryFactory.selectFrom(post)
                .where(post.title.eq(title))
                .fetch();
    }
}
```

`selectFrom`에 있는 `post`는 어디서 온 것일까? 아까 compileJava를 실행시켜서 만든 Q클래스에서 온 것이다.

이제 테스트를 해보자.

```java
@Test
void findByTitle() {
    postRepository.saveAll(Arrays.asList(
            new Post("test", "content"),
            new Post("test", "content"),
            new Post("test", "content"),
            new Post("title1", "content"),
            new Post("title2", "content"),
            new Post("title3", "content")
    ));

    final List<Post> posts = postRepositorySupport.findByTitle("test");

    assertAll(
            () -> assertThat(posts).hasSize(3),
            () -> assertThat(posts.get(0).getTitle()).isEqualTo("test")
    );
}
```

**2. Spring Data Jpa Custom Repository 적용**

위와 같이 사용하면 항상 2개의 Repository(QueryDSL의 Custom Repository, JpaRepository를 상속한 Repository)를 의존성으로 받아야 한다.

이번에는 Custom Repository를 JpaRepository 상속 클래스에서 사용해보자.

**CustomizedPostRepository**

```java
public interface CustomizedPostRepository {
    List<Post> findByTitle(final String title);
}
```

**CustomizedPostRepositoryImpl**

```java
public class CustomizedPostRepositoryImpl implements CustomizedPostRepository {

    private final JPAQueryFactory jpaQueryFactory;

    private CustomizedPostRepositoryImpl(final JPAQueryFactory jpaQueryFactory) {
        this.jpaQueryFactory = jpaQueryFactory;
    }

    @Override
    public List<Post> findByTitle(final String title) {
        return jpaQueryFactory.selectFrom(post)
                .where(post.title.eq(title))
                .fetch();
    }
}
```

**PostRepository**

```java
public interface PostRepository extends JpaRepository<Post, Long>, CustomizedPostRepository {
}
```

이렇게 구성하면 `CustomizedPostRepositoryImpl`의 코드를 사용할 수 있다. `PostRepository`는 어떻게 `CustomizedPostRepository`을 상속받아서 `CustomizedPostRepositoryImpl`의 코드를 사용할 수 있을까?

[Spring 공식 문서](https://docs.spring.io/spring-data/jpa/docs/2.1.3.RELEASE/reference/html/#repositories.custom-implementations)를 보자. 요약하면 `CustomizedRepository` 인터페이스를 상속한 `Impl` 클래스의 코드를 당신의 `Repository`에 `CustomizedRepository`를 상속받아 사용할 수 있다고 한다. `CustomizedRepository`의 이름을 한 번 바꿔보았지만 잘 동작했다. 중요한 것은 `Impl` 접미사 같다.

> The most important part of the class name that corresponds to the fragment interface is the `Impl` postfix.
>
> [Spring 공식 문서](https://docs.spring.io/spring-data/jpa/docs/2.1.3.RELEASE/reference/html/#repositories.custom-implementations)

이제 테스트해보자.

```java
@Test
void findByTitle() {
    postRepository.saveAll(Arrays.asList(
            new Post("test", "content"),
            new Post("test", "content"),
            new Post("test", "content"),
            new Post("title1", "content"),
            new Post("title2", "content"),
            new Post("title3", "content")
    ));

    final List<Post> posts = postRepository.findByTitle("test");

    assertAll(
            () -> assertThat(posts).hasSize(3),
            () -> assertThat(posts.get(0).getTitle()).isEqualTo("test")
    );
}
```

**3. 상속/구현 없는 Repository**

QueryDSL만으로 Repository를 구현하는 방법이다.

**PostQueryRepository**

```java
@Repository
public class PostQueryRepository {

    private final JPAQueryFactory jpaQueryFactory;

    public PostQueryRepository(final JPAQueryFactory jpaQueryFactory) {
        this.jpaQueryFactory = jpaQueryFactory;
    }

    public List<Post> findByTitle(final String title) {
        return jpaQueryFactory.selectFrom(post)
                .where(post.title.eq(title))
                .fetch();
    }
}
```

테스트해보자.

```java
@Test
void findByTitle() {
    postRepository.saveAll(Arrays.asList(
            new Post("test", "content"),
            new Post("test", "content"),
            new Post("test", "content"),
            new Post("title1", "content"),
            new Post("title2", "content"),
            new Post("title3", "content")
    ));
    
    final List<Post> posts = postQueryRepository.findByTitle("test");

    assertAll(
    	    () -> assertThat(posts).hasSize(3),
    	    () -> assertThat(posts.get(0).getTitle()).isEqualTo("test")
    );
}
```
