# AWS에 Spring Boot 프로젝트 배포 2 - EC2 세팅＆배포 , 도메인 연결



## 🎈 AWS를 통해 배포해보자 - 2 <a href="#aws-2" id="aws-2"></a>

> 저번 글에서는 `AWS RDS`를 세팅하고, Spring Boot에`MySQL`을 연동해봤다.\
> 이번 글에서는 `AWS EC2`에 실제로 **프로젝트를 올려서 구동**시켜보고 거기에 **도메인을 연결**해서 보기좋은 하나의 웹사이트로 변신시켜보자!

## 💻 AWS EC2 생성 및 설정 <a href="#aws-ec2" id="aws-ec2"></a>

EC2 서비스에서 생성하기를 누르고 프리티어에 맞는 OS를 선택한다.\
![](https://media.vlpt.us/images/dohaeng0/post/06fd1886-8f0c-4443-807e-7c063fa9a0d4/image.png)\
우분투 18.04 버전으로 해볼 것이다.

뭐 건드릴 것 없이 바로 **검토 및 시작**으로 가서 **키페어 선택** 후 생성해준다.

![](https://media.vlpt.us/images/dohaeng0/post/bd80865a-4b68-46bb-8898-5dc17a80f622/image.png)\
인스턴스 하나가 생긴 것을 볼 수 있다.

> 프리티어 1년동안은 인스턴스를 실행해놔도 딱히 돈이 나갈일이 없지만 그 뒤에는 돈이 나간다.\
> 나중에 꼭 인스턴스 종료해주자. (_다시는 사용X 요금청구X 완전 삭제됨_)

#### 💻 EC2 인스턴스에 SSH 접속 - 가상 컴퓨터(EC2인스턴스)를 다뤄보자 <a href="#ec2-ssh-ec2" id="ec2-ssh-ec2"></a>

기본적으로 인스턴스를 생성하면 SSH를 사용할 수 있는 22번 포트가 열려있다.

Git bash를 사용해서 접속해보자

```
 ssh -i 키페어위치(파일드래그해서 넣기) ubuntu@퍼블릭ip주소
```

이 명령어를 치고 접속하면 되는데 키페어파일 위치는 그냥 git bash에 파일을 드래그 앤 드랍하면 자동으로 입력된다.

그리고 뒤에 퍼블릭 ip 주소는 생성한 EC2 인스턴스에 들어가서 복사하면 된다.\
![](https://media.vlpt.us/images/dohaeng0/post/26b9d8c8-5cfc-4d72-bd10-09443db7ab36/image.png)

***

SSH 접속이 정상적으로 됐으니 넘어가보자

## 💻 배포파일 빌드 및 EC2에 업로드 <a href="#ec2" id="ec2"></a>

이제 빌드를 먼저 해보자\
IntelliJ 에서는 다음과 같이 빌드하면 된다.

![](https://media.vlpt.us/images/dohaeng0/post/c48d4673-4196-44ec-907e-89991ab1e3f5/image.png)

* 오른쪽 구석에 Gradle을 눌러주고(본인 프로젝트는 Gradle로 했음)\
  build 밑에 톱니바퀴모양 build를 더블클릭해주면 빌드가 시작된다.

완료됐다면 왼쪽의 프로젝트 구조를 보자

![](https://media.vlpt.us/images/dohaeng0/post/0acb116c-bc6c-4d47-a8f9-1fcb02c9d21e/image.png)

* build 밑에 libs를 보면 `~~~.jar` 파일이 있다. 이게 빌드된 배포파일이다. EC2에 올려서 실행시키면 서버가 돌아가는 것이다.

***

배포파일을 올리기 전에 생성한 EC2 인스턴스에 자바 프로젝트 구동을 위한 JDK를 설치해주자.\
**SSH로 접속한 상태**에서 아래와 같이 명령어를 쳐서 설치하면 된다.

```
sudo apt-get update // 동기화한번 해주가
sudo apt-get install openjdk-8-jdk // 자바 8 설치
java -version //버전 확인 (나오면 설치된 것)
```

스프링부트 프로젝트를 자바 8 버전으로 했기 때문에 맞춰서 설치해준다.

다음으로 `FileZilla` 라는 프로그램을 이용해서 EC2 인스턴스로 배포파일을 전송해보자.

![](https://media.vlpt.us/images/dohaeng0/post/66876d84-a54a-4ded-8ef4-61c822f6151d/image.png)

* 맨 위 표시된 버튼을 눌르면 사이트관리자가 뜬다.
* New site를 눌러서 추가한다.

그리고 오른쪽\
![](https://media.vlpt.us/images/dohaeng0/post/071e5aa7-10ec-453a-9272-f72607e670ec/image.png)

* SFTP로 바꿔주고 호스트에는 EC2 인스턴스의 퍼블릭IP를 넣어준다.
* 사용자는 위에서 **git bash로 SSH 접속할때 @ 앞에붙인 이름**을 말한다.
* 로그온 유형은 키 파일 - **EC2 생성할 때의 그 키페어가 맞다.**
* 연결.

![](https://media.vlpt.us/images/dohaeng0/post/442007be-6648-4742-9c64-14ed26d1285c/image.png)

* 화면을 기준으로 왼쪽이 내 로컬 컴퓨터
* 오른쪽이 EC2 인스턴스 (가상 컴퓨터)이다.
* 오른쪽 리모트 사이트에 적당한 디렉터리를 하나 만들어주자

그리고 드래그 앤 드랍으로 옮겨준다.\
![](https://media.vlpt.us/images/dohaeng0/post/3e082a13-b0a7-4707-a673-cebf77a1c9a6/image.png)

git bash에서 옮긴 파일이 있는 디렉터리로 이동해서 ls 명령어를 쳐보면\
![](https://media.vlpt.us/images/dohaeng0/post/3bdac69d-bf1b-4165-bb9b-8953cd35407b/image.png)\
다음과 같이 파일이 정상적으로 옮겨진 것을 확인할 수 있다.

그리고 다시 git bash에서

```java
java -jar 파일이름.jar
```

쳐주면 익숙한 구동화면이 나오게 된다.\
![](https://media.vlpt.us/images/dohaeng0/post/f1e08d39-6c49-4c72-be43-b28c2c7b582d/image.png)

***

**퍼블릭IP:8080** 치고 접속을 해보자!\
하지만 접속이 안된다..

* 이유는 **8080 포트를 아직 열어주지 않았기 때문**이다.
* EC2 인스턴스에서 **8080 포트**를 열어주자
* 근데 **80번 포트**도 열어줘야한다.
* 기본 HTTP 연결은 **80번 포트로 들어가는데 그 요청을 8080으로 포트포워딩** 해줄 것이기 때문이다.

이전 글에서 RDS세팅할 때, 보안그룹을 설정한 것 처럼 보안그룹으로 들어가서 다음과 같이 인바운드 규칙을 편집해주자\
![](https://media.vlpt.us/images/dohaeng0/post/6d0c60f8-1e5d-49da-bea3-354065b7bf85/image.png)

그 다음 접속해보면 접속이 되는것을 확인할 수 있다.

여기까지 됐으면 **배포가 완료**된 것이다. 하지만 문제가 있다.\
**git bash를 꺼버리면 서버가 다운**된다.\
그럼 AWS에 배포한 의미가 없지않나??\
그리고 주소뒤에 붙은 8080포트도 없애고 싶다.\
바로 해결해보자

## 📲 nohup , 포트포워딩 <a href="#nohup" id="nohup"></a>

> 먼저 위에서 설명했듯이 **HTTP 기본 연결포트는 80번** 포트이다.\
> 즉, 80번 포트로 들어오는 연결을 8080으로 **포트포워딩**을 해주면\
> 뒤에 8080포트를 적지 않아도 **알아서 8080으로 연결**된다는 소리다.

바로 해보자. git bash에서 다음 명령어를 쳐주면 끝이다.

```
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

> 이 명령은 "라우팅 정보를 가지고 있는 테이블이 있는데, 앞으로 80으로 들어오는 포트번호는 8080으로 해석해라" 라고 입력해놓는 것이라고 한다.\
> <[출처블로그 링크](https://blog.naver.com/timeless947/221989275951)>

그리고 서버를 재시작 해주고 8080을 떼주고 접속해보면,\
![](https://media.vlpt.us/images/dohaeng0/post/612bad26-d3b2-43b4-886b-48bfdedcb797/image.png)\
성공이다.

***

다음으로 `nohup`을 통해서 **원격 접속이 끊어져도 백그라운드에서 알아서 돌아갈 수 있도록** 해보자

간단하게 하나의 명령어로 끝난다.\
jar 파일을 동작시킬 때의 명령어의 앞, 뒤에 `nohup`, `&(and표시)`만 붙여주면 된다. 다음과 같다.

```
nohup java -jar 파일이름.jar &
```

저렇게 실행시키고 연결돼있는 git bash를 종료해도 **서버는 알아서 돌아가고 있다.**\
서버를 종료시키려면 실행되고 있는 프로세스를 `kill 명령`으로 죽여주면 된다.

```
ps -ef | grep java
```

위 명령어를 치면 실행되고 있는 프로세스 중에서 java가 들어간 놈들을 다음과 같이 보여준다.\
![](https://media.vlpt.us/images/dohaeng0/post/b1434ce1-9bab-4a68-81f3-bdf3ead140fd/image.png)\
저기 표시된 19221을 종료시키면 된다.

```
kill -9 19221
```

명령으로 kill 해준다면 서버가 종료된다.

여기까지 됐으니 마지막으로 깔끔하게 도메인을 연결시켜 보도록 해야겠다. 내 페이지만의 주소를 갖는 것이다.

***

## 🎫 도메인 연결하기 <a href="#undefined" id="undefined"></a>

먼저 도메인을 구매해야 한다.\
가비아 라는 사이트가 있는데 이벤트도 하고 저렴한 도메인이 많으니 구매하기 좋은 것 같다.\
<[가비아 링크](https://www.gabia.com)>

구매했으면 관리 툴에 들어간다.\
![](https://media.vlpt.us/images/dohaeng0/post/cacf1299-ae21-460c-a0c0-e8e7e76e9a73/image.png)

다음으로 DNS정보에서 도메인 연결 설정으로 들어간다.\
![](https://media.vlpt.us/images/dohaeng0/post/0e7f2c50-d392-48d9-9bfb-4ccc47c28a6d/image.png)

또 설정으로 들어가준다..\
![](https://media.vlpt.us/images/dohaeng0/post/c6d1bcc7-c73a-45fa-9e15-4cf36ff2a822/image.png)

그 다음, DNS 설정에서 레코드 수정을 누르고\
![](https://media.vlpt.us/images/dohaeng0/post/f895cb22-0055-49c1-86fe-37eceb9c1118/image.png)\
EC2인스턴스의 퍼블릭 IP같은 정보를 입력하고 저장.\
그럼 연결이 완료된다. (약 5\~10분 대기)

그리고 서버가 돌아가고 있는 상태에서 연결한 도메인으로 접속을 시도해본다.

![](https://media.vlpt.us/images/dohaeng0/post/057e54f0-3f9d-408c-a754-5c919e3b5e00/image.png)
