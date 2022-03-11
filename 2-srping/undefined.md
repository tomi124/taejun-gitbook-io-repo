# 전체적으로 모아볼까

## Spring? <a href="#spring-3f" id="spring-3f"></a>

자바 엔터프라이즈 어플리케션 개발에 사용되는 어플리케션 프레임워크.

#### 어플리케이션의 기본 틀: 스프링 컨테이너 <a href="#ec-96-b4-ed-94-8c-eb-a6-ac-ec-bc-80-ec-9d-b4-ec-85-98-ec-9d-98-ea-b8-b0-eb-b3-b8-ed-8b-80-3a-ec-8a-a" id="ec-96-b4-ed-94-8c-eb-a6-ac-ec-bc-80-ec-9d-b4-ec-85-98-ec-9d-98-ea-b8-b0-eb-b3-b8-ed-8b-80-3a-ec-8a-a"></a>

스프링은 스프링 컨테이너 또는 어플리케이션 컨텍스트라 불리는 스프링 런타임 엔진을 제공. 설정정보를 참고해 어플리케이션을 구성 하는 오브젝트를 생성하고 관리한다.

#### 핵심 모델: IoC/DI, 서비스 추상화, AOP <a href="#ed-95-b5-ec-8b-ac-eb-aa-a8-eb-8d-b8-3a-ioc-2fdi-2c-ec-84-9c-eb-b9-84-ec-8a-a4-ec-b6-94-ec-83-81-ed-9" id="ed-95-b5-ec-8b-ac-eb-aa-a8-eb-8d-b8-3a-ioc-2fdi-2c-ec-84-9c-eb-b9-84-ec-8a-a4-ec-b6-94-ec-83-81-ed-9"></a>

**IoC/DI**

오브젝트 생명주기와 의존관계에 대한 프로그래밍 모델. 스프링의 근간이 되는 디자인 패턴.

* IoC(Inversion of Control): 개발자는 프레임워크에 필요한 기능을 개발. 기능의 호출 및 제어는 프레임워크에 의해 수행.
* DI(Dependency InjectIon): 객체들과의 관계를 관리할 때 사용하는 기법. 의존적인 객체를 직접 생성하는것이 아니라 외부에서 주입받는 형태. 모듈간의 결합도가 낮아지고 유연성이 높아진다.

**서비스 추상화**

특정 기술과 환경에 종속되지 않는 유연한 서비스 어플리케이션. JPA를 Maria DB에도 MySQL 에도 MSSQL 에도 사용할 수 있다.

**AOP (Aspect Oriented Programing)**

관점 지향 프로그래밍 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 모듈화하겠다는 것.\
여러 곳에 흩어진 중복되는 코드(흩어진 관심사) 를 모듈화 (하나의 코드로 묶음) 하여 부가적인 관심사로 묶고, 비즈니스 로직(핵심적 관심사) 에서 재사용을 하는 것.

### Spring Framework <a href="#spring-framework" id="spring-framework"></a>

> The Spring Framework provides a comprehensive programming and configuration model for modern Java-based enterprise applications - on any kind of deployment platform.
>
> A key element of Spring is infrastructural support at the application level: Spring focuses on the "plumbing" of enterprise applications so that teams can focus on application-level business logic, without unnecessary ties to specific deployment environments.

간단히 말하면, 자바 환경의 엔터프라이즈 급 개발에 필요한 기능들을 제공하여 개발자는 서비스 로직 개발에만 집중하게 하는 아주 편리한 도구!

#### 프레임워크 vs 라이브러리 <a href="#ed-94-84-eb-a0-88-ec-9e-84-ec-9b-8c-ed-81-ac-vs-eb-9d-bc-ec-9d-b4-eb-b8-8c-eb-9f-ac-eb-a6-ac" id="ed-94-84-eb-a0-88-ec-9e-84-ec-9b-8c-ed-81-ac-vs-eb-9d-bc-ec-9d-b4-eb-b8-8c-eb-9f-ac-eb-a6-ac"></a>

제어의 역전\\

![](<../.gitbook/assets/image (88).png>)

### Spring boot <a href="#spring-boot" id="spring-boot"></a>

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

Spring Framework 보다 한술 더 떠서 바로 실행시킬 수 있음.

#### Spring framework vs Spring boot <a href="#spring-framework-vs-spring-boot" id="spring-framework-vs-spring-boot"></a>

![](<../.gitbook/assets/image (78).png>) ![](<../.gitbook/assets/image (67).png>)

완성품 이지만 내가 입맛대로 설정할 수 있다.\
설명서는 공식문서가 최고!\
[https://docs.spring.io/spring-boot/docs/current/reference/html/](https://docs.spring.io/spring-boot/docs/current/reference/html/)

## Spring MVC <a href="#spring-mvc" id="spring-mvc"></a>

### MVC pattern <a href="#mvc-pattern" id="mvc-pattern"></a>

![](<../.gitbook/assets/image (33).png>)

소프트웨어 디자인 패턴으로 사용자에 의해 요청,처리,반환되는 프로세스를 3개로 분리시킨 모델.

* Model: 패턴의 중심 구성 요소. 어플리케이션의 데이터 혹은 그 데이터를 관리하는 로직이나 규칙.
* View: 사용자에게 보여지는 데이터
* Controller: 사용자의 요청을 처리하는 부분. model로 특정 작업을 요청하거나 view 데이터로 변환.

Spring MVC는 MVC 패턴 + 자바에서 제공하는 Servlet API를 구현하여 사용자가 좀 더 쉽게 웹 개발을 할 수 있도록 해주는 도구!

### Servlet <a href="#servlet" id="servlet"></a>

자바에서 제공하는 웹 프로그래밍 인터페이스.(구현체: GenericServlet, HttpServlet)\
![httpservlet.png](https://nhnent.dooray.com/files/3185825980384881614)

1. 사용자(클라이언트)가 URL을 클릭하면 HTTP Request를 Servlet Conatiner로 전송.
2. HTTP Request를 전송받은 Servlet Container는 HttpServletRequest, HttpServletResponse 두 객체를 생성.
3. web.xml(혹은 AbstractAnnotationConfigDispatcherServletInitializer) 은 사용자가 요청한 URL을 분석하여 어느 서블릿에 대해 요청을 한 것인지 찾음.
4. 해당 서블릿에서 service메소드를 호출한 후 클리아언트의 POST, GET여부에 따라 doGet() 또는 doPost()를 호출 (HTTP servlet).
5. doGet() or doPost() 메소드는 동적 페이지를 생성한 후 HttpServletResponse객체에 응답.
6. 응답이 끝나면 HttpServletRequest, HttpServletResponse 두 객체를 소멸.

### Servlet Container <a href="#servlet-container" id="servlet-container"></a>

구현되어있는 Servlet 객체를 생성, 초기화 호출, 종료 하는 생명 주기 및 실행 순서를 관리.\
대표적인 서블릿 컨테이너: Tomcat

스프링 mvc는 servlet의 구현체이기 때문에 servlet container(Tomcat)위에서 동작을 시켜야 한다. 이를 위해 여러가지 tomcat 설정이 필요하다.\
단! spring boot는 내장 톰캣이 있어서 바로 실행이 가능.

### DispatcherServlet <a href="#dispatcherservlet" id="dispatcherservlet"></a>

Servlet을 구현한 Spring의 구현체. Front controller으로써의 역할을 주로 수행.

#### FrontController <a href="#frontcontroller" id="frontcontroller"></a>

**일반적인 servlet에서의 컨트롤러 매핑 과정**

![](<../.gitbook/assets/image (74).png>)

* url 별 서블릿 매핑 설정을 별도로 다 해줘야 함.

**FrontController 패턴의 매핑**

![](<../.gitbook/assets/image (73).png>)

* FrontController에서 각 설정을 한번에 관리할 수 있음.
* FrontController에서 수행할 controller(handler)을 찾은 후 위임(delegate) 함.

#### 그외 추가 기능 <a href="#ea-b7-b8-ec-99-b8-ec-b6-94-ea-b0-80-ea-b8-b0-eb-8a-a5" id="ea-b7-b8-ec-99-b8-ec-b6-94-ea-b0-80-ea-b8-b0-eb-8a-a5"></a>

ViewResolver, MultipartResolver, LocalResolver, ThemeResolver...\
상세 기능 및 설명은 docs를 참고하자!

### Spring boot 실행과정 <a href="#spring-boot-ec-8b-a4-ed-96-89-ea-b3-bc-ec-a0-95" id="spring-boot-ec-8b-a4-ed-96-89-ea-b3-bc-ec-a0-95"></a>

1. ServletConatinerInitailizer의 구현체인 TomcatStarter에서 톰캣 실행.\\
2. DispatcherServletAutoConfiguration에서 DispatcherSerlvet이 bean으로 등록
   1. Servlet Container에 Servlet 등록
   2. Servlet Container 각종 필터들 등록
   3. Handler mapping(controller) 등록

![](<../.gitbook/assets/image (43).png>)

## JPA <a href="#jpa" id="jpa"></a>

Java Persistence API: 자바 진영의 ORM 기술 표준

#### 사용하는 이유? <a href="#ec-82-ac-ec-9a-a9-ed-95-98-eb-8a-94-ec-9d-b4-ec-9c-a0-3f" id="ec-82-ac-ec-9a-a9-ed-95-98-eb-8a-94-ec-9d-b4-ec-9c-a0-3f"></a>

* 생산성: 반복적인 CRUD 코드로 해결
* 유지보수: 엔티티에 필드 하나 추가되거나 할때 관리해줘야 하는 매핑이 별도로 없음.
* 패러다임 불일치 해결
* 벤더 독립성

![](<../.gitbook/assets/image (116).png>)

### ORM <a href="#orm" id="orm"></a>

Object Relation Mapping. 객체와 데이터베이스를 매핑.\
객체지향 언어와 데이터베이스 간의 패러다임 불일치를 ORM 프레임워크에서 해결하여 개발자는 개발에만 신경쓸 수 있게 함.

패러다임 불일치? 관계형 DB의 구조와 객체지향 언어간의 구조 차이.\
ex) 주문 1에 옵션(A,B,C) 객체를 저장 해야 할 경우 하나의 객체지만 주문, 옵션을 분리하여 관계형 DB에 저장해야 한다. DB에서 객체로 불러올 때도 마찬가지.

```
select o.order_id, op.opition_id from order o inner join option op on o.order_id = op.order_id where o.order_id = 1
```

| order\_id | option\_id |
| --------- | ---------- |
| 1         | A          |
| 1         | B          |
| 1         | C          |

```
@Entity
@Table(name = "order")
data class Order(
    @OneToMany(mappedBy = "orderId")
    val orderOptions: List<OrderOption> = mutableListOf()
)

@Entity
@Table(name = "option")
data class Option(
    @ManyToOne(targetEntity == Order::class)
    @JoinColumn(name="order_id")
    val order: Order
)
```

### Hibernate <a href="#hibernate" id="hibernate"></a>

자바를 위한 ORM 중 하나로 JPA 구현체.

![](<../.gitbook/assets/image (120).png>)

## 영속성 <a href="#ec-98-81-ec-86-8d-ec-84-b1" id="ec-98-81-ec-86-8d-ec-84-b1"></a>

### Entity Manager Factory <a href="#entity-manager-factory" id="entity-manager-factory"></a>

엔티티 매니저를 생성하는 클래스. 생성 비용이 커서 보통 어플리케이션 하나당 한개만 생성된다.

### Entity Manager <a href="#entity-manager" id="entity-manager"></a>

Entity의 C/U/R/D 와 같은 엔티티와 관련된 일을 처리한다.\
엔티티 매니저의 경우 여러 스레드가 접근할 시 동시성 문제가 발생할 수 있기 때문에 스레드간 공유를 하면 안된다.

![](<../.gitbook/assets/image (50).png>)

### 영속성 컨텍스트 <a href="#ec-98-81-ec-86-8d-ec-84-b1-ec-bb-a8-ed-85-8d-ec-8a-a4-ed-8a-b8" id="ec-98-81-ec-86-8d-ec-84-b1-ec-bb-a8-ed-85-8d-ec-8a-a4-ed-8a-b8"></a>

엔티티를 영구 저장하는 환경? 엔티티 정보를 가상의 환경(컨텍스트) 에 저장.

#### 생명주기 <a href="#ec-83-9d-eb-aa-85-ec-a3-bc-ea-b8-b0" id="ec-83-9d-eb-aa-85-ec-a3-bc-ea-b8-b0"></a>

* 비영속(new/transient): 영속성 컨텍스트와 전혀 관련이 없는 상태
* 영속(managed): 영속성 컨텍스트에 저장된 상태. persist, find, merge
* 준영속(detached): 영속성 컨텍스트에서 분리된 상태. detach, clear, close
  * detach: 특정 엔티티 분리
  * close: 영속성 컨텍스트 닫기
  * clear: 영속성 컨텍스트 초기화
* 삭제(removed): 삭제된 상태. remove

![](<../.gitbook/assets/image (65).png>)

* spring data jpa 에서의 엔티티매니저 생명주기는?
* spring data jpa 에서는 영속성 관리는 어떻게 동작할까?

## 연관관계 <a href="#ec-97-b0-ea-b4-80-ea-b4-80-ea-b3-84" id="ec-97-b0-ea-b4-80-ea-b4-80-ea-b3-84"></a>

### 연관관계 매핑 <a href="#ec-97-b0-ea-b4-80-ea-b4-80-ea-b3-84-eb-a7-a4-ed-95-91" id="ec-97-b0-ea-b4-80-ea-b4-80-ea-b3-84-eb-a7-a4-ed-95-91"></a>

엔티티 간의 관계를 객체로 매핑할 수 있도록 설정.

#### 방향 <a href="#eb-b0-a9-ed-96-a5" id="eb-b0-a9-ed-96-a5"></a>

단방향, 양방향

#### 다중성 <a href="#eb-8b-a4-ec-a4-91-ec-84-b1" id="eb-8b-a4-ec-a4-91-ec-84-b1"></a>

N:1, 1:1, 1:N, N:M

#### 주인 <a href="#ec-a3-bc-ec-9d-b8" id="ec-a3-bc-ec-9d-b8"></a>

객체를 양방향 연관관계로 만들면 주인을 정해야 한다.

### 다대일(N:1) <a href="#eb-8b-a4-eb-8c-80-ec-9d-bc-n-3a1" id="eb-8b-a4-eb-8c-80-ec-9d-bc-n-3a1"></a>

#### 단방향 <a href="#eb-8b-a8-eb-b0-a9-ed-96-a5" id="eb-8b-a8-eb-b0-a9-ed-96-a5"></a>

![](<../.gitbook/assets/image (76).png>)

```
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    ...
    
}
```

```
@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    ...
}
```

#### 양방향 <a href="#ec-96-91-eb-b0-a9-ed-96-a5" id="ec-96-91-eb-b0-a9-ed-96-a5"></a>

![](<../.gitbook/assets/image (124).png>)

```
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private Strimg username;
    
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
    
    ...   
}
```

```
@Entity
public class Team {
    
    @Id
    @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team") //주인 설정
    private List<Member> members = new ArrayList<Member>();
    
    ...
}
```

* 양방향 관계의 경우 주인은 1쪽일까 N쪽일까?

### 일대다(1:N) <a href="#ec-9d-bc-eb-8c-80-eb-8b-a4-1-3an" id="ec-9d-bc-eb-8c-80-eb-8b-a4-1-3an"></a>

![](<../.gitbook/assets/image (19).png>)

```
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private Strimg username;
    
    @JoinColumn(name="TEAM_ID")
    private Team team;
    
    ...   
}
```

```
@Entity
public class Team {
    
    @Id
    @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
    
    ...
}
```

* 일대다, 다대일 중 하나만 선택해야 한다면???
* fk 사용시 문제점

```
Member member1 = new Member("1");
Member member2 = new Member("2");

Team team1 = new Team("1");
team1.getMembers().add(member1);
team1.getMembers().add(member2);

em.persist(member1);	// INSERT
em.persist(member2);	// INSERT
em.persist(team1);	// INSERT, UPDATE member1.외래키, UPDATE member2.외래키

tx.commit();
```

* n+1 발생가능
* n+1 이란?

### 일대일 <a href="#ec-9d-bc-eb-8c-80-ec-9d-bc" id="ec-9d-bc-eb-8c-80-ec-9d-bc"></a>

* `@OneToOne`으로 선언
* 양방향 관계인 경우 주인이 아닌 쪽에 `mappedBy` 정의

### 다대다 <a href="#eb-8b-a4-eb-8c-80-eb-8b-a4" id="eb-8b-a4-eb-8c-80-eb-8b-a4"></a>

* \`@ManyToMany 로정의
* 관계형 DB 에서는 테이블 두개로 다대다 관계를 표현할 수 없음. 따라서 중간에 매핑 테이블을 둬야함.\\

![](<../.gitbook/assets/image (12).png>)

```
@Entity
public class Member {
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    
    private String username;
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
    			joinColumns = @JoinColumn(name = "MEMBER_ID"),
                	inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<Product>();
    
    ...
}

@Entity
public class Product {
    @Id
    @Column(name = "PRODUCT_ID")
    private String id;
    
    private String name;
    
    @ManyToMany(mappedBy = "products")
    private List<Member> members;
    ...
    
    
 }
```

* 거의 사용하지 않음!
* `ManyToMany` 보단 매핑테이블을 엔티티로 만들고, `OneToMany` - `ManyToOne` 관계로

## Spring Data JPA <a href="#spring-data-jpa" id="spring-data-jpa"></a>

### 편리한 spring jpa 쿼리메소드 <a href="#ed-8e-b8-eb-a6-ac-ed-95-9c-spring-jpa-ec-bf-bc-eb-a6-ac-eb-a9-94-ec-86-8c-eb-93-9c" id="ed-8e-b8-eb-a6-ac-ed-95-9c-spring-jpa-ec-bf-bc-eb-a6-ac-eb-a9-94-ec-86-8c-eb-93-9c"></a>

[https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods)\
![image.png](https://nhnent.dooray.com/files/3185825985262344950)

### JPQL <a href="#jpql" id="jpql"></a>

`@Query`를 이용하여 객체지향 쿼리를 직접 작성할 수 있다.

```
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.firstname like %?1")
  List<User> findByFirstnameEndsWith(String firstname);
}
```

#### native query <a href="#native-query" id="native-query"></a>

객체지향 기반이 아닌 실제 SQL을 작성할수도 있다. 만약 여러 datasource를 사용한다면 사용에주의해야 한다.

```
public interface UserRepository extends JpaRepository<User, Long> {

  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
}
```

### QueryDSL <a href="#querydsl" id="querydsl"></a>

복잡하거나, 쿼리문을 동적으로 변경시킬 수 있는 쿼리.\
쿼리를 java 코드로 작성 가능하여 동적인 쿼리를 작성할 수 있다.

\\
