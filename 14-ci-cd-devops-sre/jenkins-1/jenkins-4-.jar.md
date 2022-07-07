# Jenkins 4 - .jar배포 (깃헙)

**1. Jenkins관리 - 플러그인 관리 -> 설치 -> Publish Over SSH 검색 후 설치**

**2. Jenkins관리 - 시스템 설정 -> Publish Over SSH**

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F86e270fa-c55d-431a-93a5-cae0bfa754d4%2Fimage.png)

**3. 이전에 만들었던 아이템 test의 구성 클릭**

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Feaf2fbe1-fa0e-46ba-8bf0-57f5d7102796%2Fimage.png)

**4. 빌드 후 조치 클릭 -> 빌드 후 조치 추가 (Send build artifacts over SSH 선택)**

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Ff76cc777-cb7d-4c9e-bb00-50121ea215b8%2Fimage.png)\
위 이미지 처럼 작성 후 저장

**5. ec2 로 접속해서 /home/ubuntu/app/script/에서 test.sh 생성**

```shell
REPOSITORY=/home/ubuntu/app
cd $REPOSITORY/project/ 
echo "> now ing app pid find!" 
CURRENT_PID=$(pgrep -f project) 
echo "$CURRENT_PID" 
if [ -z $CURRENT_PID ]; then 
	echo "> no ing app." 
else 
	echo "> kill -9 $CURRENT_PID" 
    kill -9 $CURRENT_PID 
    sleep 3 
fi 
echo "> new app deploy" 
JAR_NAME=$(ls $REPOSITORY/ |grep 'project' | tail -n 1) 
echo "> JAR Name: $JAR_NAME" 
nohup java-jar $REPOSITORY/$JAR_NAME &
```

**6. build 를 하면 됨!**

![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F3fde8bfc-141b-415c-b7d0-5123109b9d14%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-08-22%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.37.45.png)\
ec2에서 해당 경로로 가서 jar가 생겼는지 확인하면 됨!

**github push 시 자동 빌드**

[https://goddaehee.tistory.com/260?category=399178](https://goddaehee.tistory.com/260?category=399178)\
참고\
\
출처 : [https://velog.io/@jae\_cheol/Jenkins-.jar%EB%B0%B0%ED%8F%AC-%EA%B9%83%ED%97%99](https://velog.io/@jae\_cheol/Jenkins-.jar%EB%B0%B0%ED%8F%AC-%EA%B9%83%ED%97%99)
