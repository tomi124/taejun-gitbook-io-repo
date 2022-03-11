# AWS에 Spring Boot 프로젝트 배포 1 - RDS, MySQL 세팅

## 🎈 AWS를 통해 배포해보자 <a href="#aws" id="aws"></a>

> 이번에는 나눠서 작성을 할 것인데 이어지는 글에서는 만든 프로젝트를 `AWS` 를 통해 **배포** 해볼 것이다.\
> 그리고 **여태까지는 `H2` 데이터베이스**를 통해 테스트용으로만 사용했지만,\
> 이제는 **서버가 다운돼도 데이터가 초기화되지 않게 `MySQL` 을 연동**해 볼 것이다.\
> 나 혼자 로컬로 접속하는 것이 아닌 다른 사람들도 내 페이지에 접속해서 사용할 수 있게 되는 것이다!

## 🔧 AWS RDS 설정 <a href="#aws-rds" id="aws-rds"></a>

AWS RDS 서비스에 접속해서 **데이터베이스를 먼저 생성**해보자\
![](https://media.vlpt.us/images/dohaeng0/post/da8efaa3-b275-4604-8614-9f12ceadda86/image.png)

\


***

![](https://media.vlpt.us/images/dohaeng0/post/ad9aa6db-79b8-4cbf-8813-a335850a9c2b/image.png)

* `표준생성` 선택

***

![](https://media.vlpt.us/images/dohaeng0/post/315ab6ff-575c-4135-854e-66685819cedb/image.png)

* `MySQL` 선택

***

![](https://media.vlpt.us/images/dohaeng0/post/0bac59b0-fa58-4811-a812-fbafa2c5f666/image.png)

* 프리티어 이용자들은 꼭!! `프리티어` 선택
* 나머지는 돈 많이 나감

***

![](https://media.vlpt.us/images/dohaeng0/post/226ae955-a327-4c2a-9f81-43dc8a200885/image.png)

* 사용할 때, 알아보기 위한 식별자와 마스터 이름, 암호가 필요하다.
* 까먹지 않도록 본인이 기억할 수 있는 것으로 하기

***

![](https://media.vlpt.us/images/dohaeng0/post/b4831449-f65c-4df7-8f56-5f8ac91b714f/image.png)

* **퍼블릭 액세스**를 반드시 `"예"`로 변경해줘야 한다. 그래야 접속을 할 수 있다.
* 보안그룹도 새로 설정할 것이기 때문에 만들어준다.
* 가용영역은 아무거나. 포트도 그대로.

***

![](https://media.vlpt.us/images/dohaeng0/post/5db4e155-d6c0-47c5-ab90-631447e61299/image.png)

* 추가 구성에서 초기데이터베이스 이름 꼭 설정하기
* 지정하지 않으면 생성하지 않는다고 한다.

\
그러고 나서 생성하기를 눌러주면 된다. 그럼 아래와 같이 새 데이터베이스가 생성된 것을 볼 수 있다.

![](https://media.vlpt.us/images/dohaeng0/post/ec724dae-e7b4-4451-8f69-ab923489c415/image.png)

만들어진 데이터베이스를 클릭하고 들어가서 **보안그룹 설정**을 해보자

![](https://media.vlpt.us/images/dohaeng0/post/309a5806-a626-4b7d-9268-0db21f7cbdf2/image.png)

* VPC 보안그룹 클릭해서 이동

***

![](https://media.vlpt.us/images/dohaeng0/post/3a883730-6c29-4199-a7fc-4127967a0697/image.png)

* 보안 그룹 ID 클릭해서 이동

***

![](https://media.vlpt.us/images/dohaeng0/post/9d06d8f5-1ec9-4ced-aeb4-bb52427a3590/image.png)

* 그 다음 인바운드 규칙 편집을 해보자

***

![](https://media.vlpt.us/images/dohaeng0/post/900a2dfb-3ace-42b5-a26d-4c2612a8955f/image.png)

* 위치무관으로 아무나 3306포트에 접근할 수 있게 열어준다.
* 어차피 마스터이름과 암호는 우리만 알고 있기 때문에 일단은 그렇게 한다. ~~_실무에서는 지정해놓는다고 하지만 연습하는 단계기 때문에_~~

***

## 🔧 Spring Boot , MySQL 연동 <a href="#spring-boot-mysql" id="spring-boot-mysql"></a>

![](https://media.vlpt.us/images/dohaeng0/post/073aa738-72c4-48b5-a6b9-60ef9ecedc93/image.png)

* 이제 프로젝트에 MySQL을 연동할건데 연결을 위해서는 DB의 엔드포인트가 필요하다.
* 저걸 복사해서 IntelliJ로 돌아가자

***

![](https://media.vlpt.us/images/dohaeng0/post/261d3295-f602-4b34-b94f-c2e314f730af/image.png)

* 오른쪽 구석에 있는 database를 클릭하면 창이 펴지는데 +를 눌러서 MySQL을 눌러주자

***

![](https://media.vlpt.us/images/dohaeng0/post/83f6d433-6894-440c-9f29-796f78cd997b/image.png)

* 입력할 것은 이 정도다.
* Name은 아무거나 해도 된다.
* Host에는 복사한 엔드포인트를 붙여넣어 준다.
* User와 Password 에는 생성할 때 설정한 마스터 이름과 암호를 입력해준다.
* Database에는 생성할 때 추가구성에서 설정한 이름을 넣어준다.

***

![](https://media.vlpt.us/images/dohaeng0/post/81a8607c-2087-49bb-8b9d-3519dafc0c09/image.png)

* 그러고 ok 해주면 데이터베이스가 들어가 있는걸 확인할 수 있다.

***

다음으로 스프링부트에 설정을 해줘야한다.

#### application.properties <a href="#applicationproperties" id="applicationproperties"></a>

```java
spring.datasource.url=jdbc:mysql://엔드포인트:3306/db이름(myselectshop)
spring.datasource.username=마스터이름
spring.datasource.password=패스워드
spring.jpa.hibernate.ddl-auto=update
```

이러면 연동은 끝이다. 서버를 실행시켜서 테스트를 해보자

![](https://media.vlpt.us/images/dohaeng0/post/e9d6d8b9-bf41-4ba8-9e72-fd58f6355577/image.png)

데이터가 아주 잘 들어가 있다. 서버를 재시작해도 그대로 남아있다.\
여기까지가 MySQL 세팅 끝이다.

다음 글에서는 AWS EC2에 프로젝트를 올려서 배포해보자



출처 : [https://velog.io/@dohaeng0/AWS%EC%97%90-Spring-Boot-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EB%B0%B0%ED%8F%AC-1-RDS-MySQL-%EC%84%B8%ED%8C%85](https://velog.io/@dohaeng0/AWS%EC%97%90-Spring-Boot-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EB%B0%B0%ED%8F%AC-1-RDS-MySQL-%EC%84%B8%ED%8C%85)
