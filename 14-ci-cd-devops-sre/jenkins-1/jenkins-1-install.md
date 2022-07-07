# Jenkins 1 - install



![post-thumbnail](https://velog.velcdn.com/images/jae\_cheol/post/dad33453-5171-4cbf-b8a3-88c4f85a5b29/image.png)

## 환경 <a href="#undefined" id="undefined"></a>

* docker ubuntu 내에 설치

## 설치 <a href="#undefined" id="undefined"></a>

1. 우분투 이미지 다운로드

```
docker pull ubuntu
```

1. 우분투 실행

```
docker run -i -t --name ubuntu ubuntu /bin/bash 
#exit 로 나오면 종료 -d 붙혀주면 백그라운드 실행
```

2-1. 종료하고 재실행 및 콘솔 들어가기

```
docker restart ubuntu # 재실행
docker attach ubuntu # 콘솔 들어가기
```

1. apt 업데이트 및 설치해야할 것들

```
apt-get update
apt-get install openjdk-8-jdk
apt-get install wget gnupg systemctl
```

1. 젠킨스 저장소 key 다운로드

```
wget -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
```

1. APT 데이터베이스에 공식 젠킨스 리포지토리 추가

```
sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fe917a7a2-0529-48f3-8ae6-4f022c0a35a1%2Fimage.png)\
위처럼 첫번째 박스처럼 나오면 다음 명령어로 해결한다.

```
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys [key]
```

1. 젠킨스 설치

```
apt update && sudo apt install jenkins -y 
```

1. 젠킨스 포트 변경

```
vi /etc/default/jenkins #서버 포트 변경
```

1. 젠킨스 실행

```
service jenkins start

systemctl status jenkins  jenkins 서비스 상태 확인

service jenkins restart
```

* 만약에 도커를 실행할 때 포트를 안열어줬으면 지금까지 한 작업 저장후 다시 실행하고 젠킨스 서비스 재실행 해주자.

```
docker stop [CONTAINER ID]
docker commit [CONTAINER ID] [저장할 이미지]
docker run -p 8080:8080 -i -t --name jenkins jenkins /bin/bash
```

1. 젠킨스 초기 비밀번호

```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F26573c17-f4a5-424d-8739-b70824e92740%2Fimage.png)

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F8732e020-6ec4-4a16-afef-797d2bb743a2%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-15%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%203.31.44.png)

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F3b45290b-78f7-48e1-aac9-6a59a92c5738%2Fimage.png)

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fd9d44f9c-bdd0-4fc5-afec-c47892971071%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-15%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%203.36.31.png)

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F53556f90-522d-4214-b360-8c1ccbdbdc42%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-15%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%203.36.37.png)



출처 : [https://velog.io/@jae\_cheol/Jenkins-%EC%84%A4%EC%B9%98](https://velog.io/@jae\_cheol/Jenkins-%EC%84%A4%EC%B9%98)
