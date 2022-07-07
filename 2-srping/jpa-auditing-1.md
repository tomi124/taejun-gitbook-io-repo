# JPA Auditing 사용하기

![](https://media.vlpt.us/images/shawnhansh/post/5b15e743-f666-4827-995f-8021ec89a9b1/michael-dziedzic-uZr0oWxrHYs-unsplash.jpg)

> 해당 내용은 [이동욱님](https://jojoldu.tistory.com/) 저서 '[스프링 부트와 AWS로 혼자 구현하는 웹 서비스](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR\&mallGb=KOR\&barcode=9788965402602)'를 공부하며 정리한 내용입니다.

생성시간과 수정시간을 처리하기 위해 단순하고 반복적인 코드가 모든 테이블과 서비스 메서드에 포함된다.\
JPA Auditing을 사용하면 이 문제를 해결할 수 있다.

### BaseTimeEntity <a href="#basetimeentity" id="basetimeentity"></a>

```java
import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass   
@EntityListeners(AuditingEntityListener.class)  
public class BaseTimeEntity {

    @CreatedDate    
    private LocalDateTime createdDate;

    @LastModifiedDate   
    private LocalDateTime modifiedDate;

}
```

모든Entity의 상위 클래스가 되어 Entity들의 createdDate, modifiedDate를 자동으로 관리

#### @MappedSuperclass <a href="#mappedsuperclass" id="mappedsuperclass"></a>

* JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 이 클래스의 필드도 컬럼으로 인식하게 된다

#### @EntityListeners(AuditingEntityListener.class) <a href="#entitylistenersauditingentitylistenerclass" id="entitylistenersauditingentitylistenerclass"></a>

* BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다

#### @CreatedDate <a href="#createddate" id="createddate"></a>

* Entity가 생성되어 저장될 때 시간이 자동 저장된다

#### @LastModifiedDate <a href="#lastmodifieddate" id="lastmodifieddate"></a>

* 조회한 Entity의 값을 변경할 때의 시간이 자동 저장된다.

이후에는 Entity에서 BaseTimeEntity를 extends 시키고, Application 클래스에 @EnableJpaAuditing 어노테이션을 추가한다.

```java
...
@Entity
public class Posts extends BaseTimeEntity {

...
```

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### @EnableJpaAuditing <a href="#enablejpaauditing" id="enablejpaauditing"></a>

* JPA Auditing 활성화

### PostsRepositoryTest <a href="#postsrepositorytest" id="postsrepositorytest"></a>

```java
import com.shawn.springboot.domain.posts.Posts;
import com.shawn.springboot.domain.posts.PostsRepository;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest    
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void BaseTimeEntity_등록(){
        //given
        LocalDateTime now = LocalDateTime.of(2019,6,4,0,0,0);
        postsRepository.save(Posts.builder()
                .title("test_title")
                .content("test_content")
                .author("author")
                .build());

        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);

        System.out.println(">>>>>> createDate : "+posts.getCreatedDate() +", modifiedDate : " + posts.getModifiedDate());

		assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }

```

출처 : [https://velog.io/@shawnhansh/SpringBoot%EC%97%90%EC%84%9C-JPA-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B04-JPA-Auditing](https://velog.io/@shawnhansh/SpringBoot%EC%97%90%EC%84%9C-JPA-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B04-JPA-Auditing)
