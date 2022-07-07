# Jenkins 5 - EC2 도커에서 젠킨스 실행하기



```
 chmod 400 /Users/thor/Desktop/jenkins.pem
  ssh -i ~/Desktop/jenkins.pem ubuntu@[IP Address]
```

1. apt가 저장소를 사용할 수 있게 함

```
sudo apt-get install apt-transport-https ca-certificates
```

1. 도커 공식 GPG 키 등록

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F33d72c3d-f7a0-410b-9a65-99481e3eb1d3%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-10-25%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.27.48.png)

1. 공식 저장소 추가

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

1. apt 업데이트

```
sudo apt update
```

1. 도커 설치

```
sudo apt-get install docker-ce
```

1. 도커 확인

```
sudo docker info
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fc31ce9fe-1ad7-4c1a-b98e-cd4cc3567a0d%2Fimage.png)

1. 젠킨스 설치 및 실행

```
 docker run -d --name jenkins_dev -p 8080:8080  jenkins/jenkins:lts
```

1. 젠킨스 컨테이너 접속

```
docker exec -it jenkins_dev bash
```

* \[IP ADDRESS]:8080 접속 시 초기 패스워드 위치를 가르쳐줌\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F16184340-d49a-440c-be3d-f41e86ad75f1%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-10-25%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.45.42.png)

1. 젠킨스 초기 비밀번호

```
cat /var/jenkins_home/secrets/initialAdminPassword
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F396dfc6f-2651-4e7a-ad3d-2f4aaf4e760a%2Fimage.png)

### 데이터 볼륨을 이용한 젠킨스 컨테이너 실행 <a href="#undefined" id="undefined"></a>

* 컨테이너를 삭제하게 되면 jenkins\_home 폴더도 같이 삭제가 된다.
* 데이터 볼륨을 이용해 젠킨스를 도커 위에서 수행하는데 더 좋은 방식이 있음.
* 데이터 볼륨이란? 데이터가 컨테이너의 라이프 사이클에 상관없이 영구적으로 저장하는 특정 폴더이다.

1. 젠킨스 컨테이너 실행

```
docker run -d --name jenkins_prod -p 8080:8080 -p 50000:50000 -v jenkins-home-prod:/var/jenkins_home jenkins/jenkins:lts
```

* \-v jenkins-home-prod:/var/jenkins\_home 옵션은 jenkins-home-prod라는 이름으로 데이터 볼륨을 만들어 /var/jenkins\_home 폴더에 연결한다.

1. jenkins\_prod 컨테이너 내의 /var/jenkins\_home 폴더의 내용을 보기 위해 다음 명령어를 실행

```
 docker exec -it jenkins_prod ls -lrt /var/jenkins_home
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F112ef278-cef3-4a3b-bd55-1ef687ff1d22%2Fimage.png)\
3\. 도커 볼륨 목록 확인

```
docker volume ls
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F3c494191-273d-4a1f-8a4b-00dc35c20e69%2Fimage.png)\
4\. 이제 영구적인 jenkins\_home 폴더를 가진 젠킨스 도커 컨테이너가 생성됬음.

1. 도커 젠킨스에 접속 후 최소 비밀번호를 가져온다

```
 docker exec -it jenkins_prod bash
 cat /var/jenkins_home/secrets/initialAdminPassword
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F555cba8d-f29e-4052-89e4-96c2fe652881%2Fimage.png)\
6\. 초기 접속시 화면\
![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Ffcc4dc69-53ba-45b0-928a-46343cf61f17%2Fimage.png)

* 위에서 확인한 비밀번호 입력 후 Continue 클릭

1. 다음 페이지로 Custimize Jenkins 화면이 나옴(Install suggested plugins 클릭)\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F779dcf3d-5f93-4b3a-8491-38e0922d8009%2Fimage.png)
2. 위 과정에서 설치가 끝나면 어드민 계정을 만들라는 페이지가 나옴\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F33ac945c-b759-4a5e-9765-30d3d7dfbf22%2Fimage.png)
3. 다음 페이지로 그냥 Save And Finish 버튼 클릭\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F9a59fe9d-e96d-4518-ba76-6ae4ba14084f%2Fimage.png)
4. 환경 설정 끝!!
5. /var/jenkins\_home/users 폴더내에 모든 사용자 정보가 있음
6. 이제 jenkins\_prod 컨테이너를 삭제해보자

```
docker kill jenkins_prod
docker rm jenkins_prod
```

1. 도커 목록확인

```
docker ps -a
```

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F88557bb6-9081-4e72-9521-4b7fc2c8b53f%2Fimage.png)

* 컨테이너가 존재하지 않는 것을 볼 수 있음.

1. 볼륨 확인\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F179fa0d1-67ab-4ed3-8075-67b52b564846%2Fimage.png)

* 컨테이너는 삭제됬지만 볼륨은 남아있다.

1. jenkins-home-prod 볼륨을 사용하는 새로운 젠킨스 컨테이너를 생성하자

```
docker run -d --name jenkins_prod -p 8080:8080 -p 50000:50000 -v jenkins-home-prod:/var/jenkins_home jenkins/jenkins:lts
```

1. 다시 해당 페이지로 접속을 해보면 세팅페이지가 아닌 로그인페이지로 이동을 한다.\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F516b81ee-e702-4b59-af06-cb3b21de127a%2Fimage.png)



출처 : [https://velog.io/@jae\_cheol/Jenkins-EC2-%EB%8F%84%EC%BB%A4%EC%97%90%EC%84%9C-%EC%A0%A0%ED%82%A8%EC%8A%A4-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0](https://velog.io/@jae\_cheol/Jenkins-EC2-%EB%8F%84%EC%BB%A4%EC%97%90%EC%84%9C-%EC%A0%A0%ED%82%A8%EC%8A%A4-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0)
