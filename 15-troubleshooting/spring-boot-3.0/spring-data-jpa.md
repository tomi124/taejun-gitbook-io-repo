# Spring data jpa

### **개요**

* Spring Data JPA를 업그레이드 하는 과정, 또는 업그레이드 한 이후에 발견된 이슈와 해결 방안을 정리한다.

### **종속성 분석**

* [JDK 17](https://docs.oracle.com/javase/specs/jls/se17/html/index.html) ([PDF](https://docs.oracle.com/javase/specs/jls/se17/jls17.pdf))
* [Jakarta Persistence 3.1](https://jakarta.ee/specifications/persistence/3.1/jakarta-persistence-spec-3.1.html) ([PDF](https://jakarta.ee/specifications/persistence/3.1/jakarta-persistence-spec-3.1.pdf)) / Jakarta EE Platform 10
* [Spring Boot 3.0.1](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/) (org.springframework.boot:spring-boot:3.0.1)
  * Spring Boot Starter Data JPA 3.0.1 (org.springframework.boot:spring-boot-starter-data-jpa:3.0.1)
    * [**Spring Data JPA 3.0.0**](https://docs.spring.io/spring-data/jpa/docs/3.0.0/reference/html/) (org.springframework.data:spring-data-jpa:3.0.0)
      * [Spring Data](https://docs.spring.io/spring-data/commons/docs/current/reference/html/) 2022.0.0
      * Jakarta Annotation API 2.1.1 (jakarta.annotation:jakarta.annotation-api:2.1.1)
      * Spring ORM 6.0.3 (org.springframework:spring-orm:6.0.3)
    * [**Hibernate 6.1.6**](https://docs.jboss.org/hibernate/orm/6.1/userguide/html\_single/Hibernate\_User\_Guide.html) (org.hibernate.orm:hibernate-core:6.1.6.Final)
      * Jakarta Persistence API 3.1.0 (jakarta.persistence:jakarta.persistence-api:3.1.0)
      * Jakarta Transaction API 2.0.1 (jakarta.transaction:jakarta.transaction-api:2.0.1)

### **Hibernate 6에서 대체 뭐가 달라졌길래?**

* 전체적인 변경사항에 대해서는 [Hibernate 6.0 Release Announcement](https://in.relation.to/2022/03/31/orm-60-final/)과 [Migration Guide](https://docs.jboss.org/hibernate/orm/6.0/migration-guide/migration-guide.html#logical-1-1-unique)를 참고하라.

#### @OneToOne 매핑 컬럼에 UNIQUE 제약 조건 자동 생성

* 이전 버전에서는 @OneToOne 매핑된 컬럼에 대해 데이터베이스 UNIQUE 제약 조건을 만들지 않았지만, 6.2 버전부터는 UNIQUE 제약 조건이 자동으로 생성된다. (_Hibernate DDL 스키마 자동 생성 기능으로 테이블을 생성하는 경우에 해당_)
  * 만약 매핑된 엔티티에 대해 논리적 Soft-delete 기능을 사용중이라면 (ex. is\_used, deleted 등의 비활성화/삭제 플래그를 사용하고 있는 경우) 해당 컬럼에 중복 값이 존재할 수도 있기 때문에 Hibernate DDL 자동 생성을 이용할 때 문제가 발생할 수 있다.
* 애플리케이션에 문제가 발생하는 경우 @jakarta.persistence.ForeignKey(NO\_CONSTRAINT)를 사용하여 UNIQUE 제약 조건 생성을 건너 뛸 수 있다.
* 또는, 기존의 @OneToOne 매핑을 @ManyToOne + @UniqueConstraint로 변경할 수도 있다.

#### **SQL을 생성하는 내부 방식이 달라졌어요**

* Hibernate 5까지는 Criteria API가 먼저 JPQL/HQL로 변환된 후, 이로부터 SQL을 직접 생성하는 구조였다.
* Hibernate 6부터는 SQL을 AST(Abstract Syntax Tree)형태로 추상화한 SQM(Semantic Query Model)이 도입되었다. Criteria API든 JPQL/HQL이든 먼저 SQM으로 변환되고, SQM 모델은 일정한 규칙에 따라 SQL 구문으로 변환된다.
* org.hibernate.orm.query.sqm.ast 로거의 로깅 레벨을 debug로 바꾸면 SQL AST가 어떻게 만들어지는지 로그로 확인할 수 있다.
* 하지만 SQM의 신규 도입으로 인해 SQM을 만드는 과정, 또는 SQM을 SQL로 변환하는 과정이 아직 안정화되지 않은 것으로 보인다. 이에 따라 생성된 SQL이 기대한 SQL과 다른 여러 케이스가 발견되고 있는 상황이다.
* **한 줄 요약: Hibernate 6로 오면서 SQL을 생성하는 내부 로직이 완전히 변경되었고, 아직까지 안정화 되지 않아 여러 버그가 존재한다.**
* [참고자료](https://vladmihalcea.com/hibernate-sqm-semantic-query-model/)

### **이슈**

* 만약 Hibernate의 버그 또는 이슈라고 생각되는 경우 [Hibernate ORM Issue Board](https://hibernate.atlassian.net/jira/software/c/projects/HHH/issues/?filter=allissues)에서 비슷한 이슈가 리포트 된 것이 있는지부터 확인해보자.

#### @Where를 사용한 조인 쿼리 조건이 원하는 대로 만들어지지 않음

* 비슷한 이슈: [@OneToMany relationship with @Where on child table generates wrong sql](https://hibernate.atlassian.net/browse/HHH-15902) (HHH-15902)
  * 2022-12-17 리포트 된 이후 아직 해결되지 않은 이슈
* 만들어지는 SQL문의 조건절 위치가 달라져서, 원하던 LEFT OUTER JOIN이 아닌 INNER JOIN처럼 동작하게 되어 버림





**데모 프로젝트**

* 정말 Hibernate 6.1의 문제인 것인지 확인해 보기 위해서, 기타 다른 종속성 없이 Hibernate Core만 가지고 데모 프로젝트를 개발하여 테스트 해 보았다.



```
select
    member0_.id as id1_1_0_,
    member0_.name as name2_1_0_,
    memberinfo1_.id as id1_0_1_,
    memberinfo1_.active as active2_0_1_,
    memberinfo1_.city as city3_0_1_,
    memberinfo1_.member_id as member_i4_0_1_ 
from
    members member0_ 
left outer join
    member_information memberinfo1_ 
        on member0_.id=memberinfo1_.member_id 
        and (
            memberinfo1_.active = 1
        )  
where
    member0_.id=?
```

```
select
    m1_0.id,
    i1_0.id,
    i1_0.active,
    i1_0.city,
    i1_0.member_id,
    m1_0.name 
from
    members m1_0 
left join
    member_information i1_0 
        on m1_0.id=i1_0.member_id 
where
    m1_0.id=?
```

* 테스트 방법: docker-compose 이용하여 로컬 MySQL 5.7 띄워둔 후 Application.kt의 main() 함수 실행
* 테스트 결과: Hibernate 5.6 → 6.1 업그레이드 이후 **Fetch Join 방식과 Entity Graph 방식 조회시 만들어지는 SQL 구문이 달라졌다는 것, 그리고 그로 인해 조회 결과가 달라졌다는 것**을 확인할 수 있었다.
  * **구체적으로는, Entity Graph를 이용해서 @OneToOne 연관관계를 fetch Join 하는 경우에 @Entity 클래스에 붙인 @Where clause가 무시되는 것으로 보인다.**

#### **@DiscriminatorValue를 사용한 엔티티의 조인 쿼리가 잘못 만들어짐**

* 관련 문서: [Hibernate 6 Wrong query generated with @DiscriminatorValue](https://discourse.hibernate.org/t/hibernate-6-wrong-query-generated-with-discriminatorvalue/6984)
  * Stackoverflow: [Hibernate 6 Wrong query generated](https://stackoverflow.com/questions/74695210/hibernate-6-wrong-query-generated)
* 2022-12-06 리포트 된 이후 아직 해결되지 않은 이슈

```
SELECT *
  FROM users user0_
  LEFT OUTER JOIN udf_value properties14_ ON user0_.id = properties14_.entity_id
      AND properties14_.target_type = 1
  WHERE user0_.id = ?
```

```
 SELECT *
  FROM users u1_0
  LEFT JOIN udf_value p2_0 ON u1_0.id = p2_0.entity_id
  WHERE p2_0.target_type = 1
      AND u1_0.id = ?
```

* 조인의 ON 절에 들어가던 조건이 바깥 WHERE 문으로 빠져나온다는 점에서 @Where를 사용한 쿼리가 잘못 만들어지는 이슈와 매우 유사하다. (_함께 fix될 가능성도 있을 수 있겠다._)



### Useful Articles

* Spring Data 공식 예제 코드는 [spring-data-examples GitHub Repository](https://github.com/spring-projects/spring-data-examples)를 참고하자.
* Hibernate 공식 자료들은 [Hibernate ORM GitHub Wiki](https://github.com/hibernate/hibernate-orm/wiki)를 참고하자.
* [Migrating from JPA 2.x to 3.0](https://thorben-janssen.com/migrating-jpa-2-x-to-3-0/) ([YouTube](https://www.youtube.com/watch?v=11mB8NM8g8c))
* [Kotlin으로 JPA Entity를 정의하는 방법](https://spoqa.github.io/2022/08/16/kotlin-jpa-entity.html)
