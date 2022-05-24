# JPA Auditing과 제대로 알아야 할 @PreUpdate



#### JPA Auditing?

서비스를 운영하다보면 데이터가 변경되었을 때 누가 값을 변경했고, 언제 변경했는지를 남겨야할 때가 있다. \
Spring Data Jpa 에서는 이에 대해 어노테이션으로 편리하게 Audit(감시) 기능을 제공해주고 있다. \
엔티티를 영속성 컨텍스트에 저장하거나 조회를 수행한 후 수정하게 되면 이에 대한 변경 시간 등을 자동으로 매핑하여 데이터베이스에 반영해주게 된다. \
\


#### 사용 방법

**1. @EnableJpaAuditing 추가**

해당 어노테이션을 main 메서드가 있는 애플리케이션 클래스에 추가한다.&#x20;

```
@EnableJpaAuditing
@SpringBootApplication
public class NoltoApplication {

    public static void main(String[] args) {
        SpringApplication.run(NoltoApplication.class, args);
    }
}
```

**2. 감시할 대상 Entity에 AuditingEntityListener를 등록 및 필드 정의**

해당 엔티티 클래스에 @EntityListener를 선언하고, 이 안에 AuditingEntityListener를 등록한다.&#x20;

@EntityListener?

엔티티를 DB에 적용하기 이전 이후에 커스텀 콜백을 요청할 수 있는 어노테이션으로,\
해당 엔티티 라이프 사이클 중 특점 시점에 원하는 로직을 처리할 수 있게 한다. \
JPA에서는 다음과 같은 7가지 이벤트를 지원한다.&#x20;

* @PrePersist : 새로운 엔티티에 대해 persist가 호출되기 전
* @PreUpdate : 엔티티 업데이트 작업 전
* @PreRemove : 엔티티가 제거되기 전&#x20;
* @PostPersist : 새로운 엔티티에 대해 persist가 호출된 후
* @PostUpdate : 엔티티가 업데이트된 후
* @PostRemove : 엔티티가 삭제된 후
* @PostLoad : Select조회가 일어난 직후에 실행

그럼 해당 어노테이션에 등록한 AuditingEntityListener를 까보자.

![](https://blog.kakaocdn.net/dn/da8Ss9/btrmXROx2M5/oSsLslg2ncPF67WoKRmM00/img.png)

이렇게 @PrePersist와 @PreUpdate 이벤트에 대한 메서드를 구현하고 있어 이를 감지하고 있다.&#x20;

참고로 놀토같은 경우는 엔티티 마다 중복되는 값들을 BaseEntity라는 추상 클래스에 모아놓고 이를 상속받도록 구현하였는데, \
BaseEntity에 해당 값들을 설정해주었다.&#x20;

```
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
public abstract class BaseEntity {

    @CreatedDate
    @Column(nullable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime modifiedDate;

    public boolean isModified() {
        return this.modifiedDate.isAfter(this.createdDate);
    }
}
```

우리는 생성 시간과 수정 시간을 사용하기 위해 LocalDateTime 타입인 createdDate 필드와 modifiedDate 필드를 추가하였다. \
참고로 @CreatedDate는 엔티티가 생성되어 DB에 저장될 때의 시간이 자동 저장되고, \
@LastModifiedDate는 조회한 엔티티의 값을 변경할 때 시간이 자동저장된다.&#x20;

#### 문제 - LastModifiedDate가 반영되지 않아요 @PreUpdate

놀토에서는 댓글을 의미하는 Comment라는 엔티티가 존재한다.\
이 역시 BaseEntity를 상속받아 생성시간과 수정 시간을 나타내는 createAt과 modifiedAt 필드를 가지고 있다.\
우리의 비즈니스 로직에서는 댓글 수정 요청이 오면 수정 후 그 수정 여부인 isModified가 응답값에 함께 보내야 한다.

그런데 JPA의 더티체킹으로 댓글을 수정하도록 구현하고 이 엔티티를 DTO에서 수정 여부인(isModified)를 확인하는 작업이 있는데,\
엔티티가 수정이 되었는데도 계속해서 isModified값이 false인 것이다.&#x20;

**문제의 CommentService 일부**

```
public CommentResponse updateComment(Long commentId, CommentRequest request, User user) {
    Comment findComment = findEntityById(commentId);
    findComment.checkAuthority(user, ErrorType.UNAUTHORIZED_UPDATE_COMMENT);
    notifyWhenChangedToHelper(request, findComment);
    findComment.update(request.getContent(), request.isHelper()); // 더티체킹으로 수정 반영
    return CommentResponse.of(findComment, user.isCommentLiked(findComment));
}
```

**isModified를 검증하는 테스트 실패**

![](https://blog.kakaocdn.net/dn/bCRvcl/btrmXGl162J/MJ71VRpVLWMON1pKQHtqVK/img.png)

왜일까를 한참 생각해보다가 우리가 사용하고 있는 AuditingEntityListener를 다시 보게 되고, \
해당 리스너에 있는 @PreUpdate 이벤트가 언제 발생하는지를 정확히 알게 되었을 때 문제의 원인을 찾을 수 있었다.

> We should note that the @PreUpdate callback is only called if the data is actually changed — that is if there's an actual SQL update statement to run. The @PostUpdate callback is called regardless of whether anything actually changed.\
> \- Bealung https://www.baeldung.com/jpa-entity-lifecycle-events -

**@PreUpdate는 엔티티 변경이 아닌 실제 DB에 있는 데이터가 변경되는 경우 호출**된디.\
때문에 실제 SQL 업데이트 문이 나가야 해당 콜백이 호출되는 것이다.&#x20;

해당 서비스가 끝나고 영속성 컨텍스트에 있는 엔티티들의 변경사항이 실제 DB에 반영이 될 당시에는 modifiedAt 필드가 변경이 되겠지만, 이 서비스에서 트랜잭션이 커밋되기 전 응답 DTO를 만들 때 수정된 엔티티만을 가지고 아직 반영되지 않은 modifiedAt을 비교하니 계속해서 false가 나왔던 것이다.&#x20;

JPA Auditing를 사용하면서 이 문제를 해결하기 위해서 우리는 해당 서비스에서 명시적으로 수정된 Entity를 repository의 saveAndFlush를 호출하여 업데이트 쿼리를 발생시켰다.\
이랬을때 해당 @PreUpdate 콜백이 호출되었고, modifiedAt이 변경되어 수정됨 필드를 정상적으로 확인할 수 있었다.

```
public CommentResponse updateComment(Long commentId, CommentRequest request, User user) {
    Comment findComment = findEntityById(commentId);
    findComment.checkAuthority(user, ErrorType.UNAUTHORIZED_UPDATE_COMMENT);
    notifyWhenChangedToHelper(request, findComment);
    findComment.update(request.getContent(), request.isHelper());
    Comment updatedComment = commentRepository.saveAndFlush(findComment); // 명시
    return CommentResponse.of(updatedComment, user.isCommentLiked(updatedComment));
}
```
