# Jenkins 8 - 멀티브랜치 파이프라인



* 사용자가 소스코드 저장소의 모든 브랜치에 대해 파이프라인을 자동으로 생성하게 해준다.
* 깃이나 깃허브 저장소의 브랜치 중 어떤 곳에서 변경이 발생하면 자동으로 빌드를 시작시키기 위해 설계됐다.

### 1. 깃허브 인증을 젠킨스에 추가 <a href="#1" id="1"></a>

* Credentials > System > Global credentials
  1. Add Credentials 클릭\
     ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fda9c4ed8-93d2-4f63-8ead-3ec19ddcf223%2Fimage.png)
  2. Kind > Secret text 선택\
     ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fe73a3481-7434-4725-9323-355cabd1cf87%2Fimage.png)
  3. 위처럼 값 입력 후 OK 클릭\
     \

* 깃허브에서 시크릿 키 발급
  1. Setting > Developer settings > Personal access tokens
  2. 다음과 같이 체크박스 체크 후 발급받기\
     ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fdead5ac7-c54b-4cd0-9c00-d646bd9ae7e5%2Fimage.png)

### 2. 젠킨스에 깃허브 Webhooks 설정 <a href="#2-webhooks" id="2-webhooks"></a>

* Jenkins 관리 > 시스템 설정 > GitHub > Add GitHub Server\
  Credentials 부분은 위에서 만든 Credentials를 선택하고 테스트 버튼 클릭\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F3ef84820-d57f-404d-aefa-84e29a74fd68%2Fimage.png)
* 테스트가 성공하면 저장 버튼 클릭

### 3. 새로운 깃허브 저장소 만들기 <a href="#3" id="3"></a>

* [https://github.com/jglick/simple-maven-project-with-tests](https://github.com/jglick/simple-maven-project-with-tests)
* 위의 저장소를 Fork 할것임!!\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F14d40bff-7276-435a-be48-5cdc59044449%2Fimage.png)

### 4. Jenkinsfiles 이용하기 <a href="#4-jenkinsfiles" id="4-jenkinsfiles"></a>

* 위 저장소에서 포크한 레포지토리에서 Jenkinsfiles를 다음과 같이 작성 후 커밋!

```
node('master') {
  checkout scm
  stage('Build') {
    withMaven(maven: 'M3'){
      if(isUnix()){
        sh 'mvn -Dmaven.test.failure.ignore clean package'
      }
      else {
        bat 'mvn -Dmaven.test.failure.ignore clean package'
      }
    }
  }
  stage('Results') {
    junit '**/target/surefire-reports/TEST-*.xml'
    archive 'target/*.jar'
  }
}
```

### 5. 젠킨스에서 멀티브랜치 파이프라인 생성하기 <a href="#5" id="5"></a>

* 대시보드 > New Item > multibranch-pipeline 선택\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F09f9e866-764e-45b8-9823-28966e74b77d%2Fimage.png)
* Branch Sources 탭 클릭 > Add Source 클릭 > Github을 선택하면 아래 이미지처럼 새로운 영역이 생김\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fb9c081a9-9662-4478-ac87-2d52b4f52335%2Fimage.png)
* Credentials 는 만들어 둔것을 선택하고\
  (Secret text는 목록에서 안뜸. 이거를 해결해야할 듯.. 그래야 7번 테스트가 잘될듯함.)\
  Repository는 아래 이미지처럼 입력 및 선택한다\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F22261822-c926-4e74-8c45-ac9401ef8451%2Fimage.png)
* 저장 클릭!

### 6. Webhooks 재등록 <a href="#6-webhooks" id="6-webhooks"></a>

1. Jenkins 관리 > 시스템 설정 > Github 영역에서 두번째 고급 버튼 클릭\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fad60b720-87ad-46b9-9574-6f7c8ff2a44d%2Fimage.png)
2. Re-register hooks for all jobs 버튼 클릭\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F21468c8f-9435-4f93-9ce0-aa67490d8e80%2Fimage.png)
3. 해당 레포지토리 > Settings > Webhooks 탭\
   아래의 이미지처럼 웹훅스가 등록된 것을 볼 수 있다.\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fa756b1c5-67aa-426b-922c-8053a26bd319%2Fimage.png)

### 7. 멀티브랜치 파이프라인 테스트 <a href="#7" id="7"></a>

* 레포지토리에서 feature라는 브랜치 생성하면 젠킨스 파이프라인은 즉시 실행함.\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F9aea3cad-2233-4248-93d7-ba5b649f0702%2Fimage.png)

출처 : [https://velog.io/@jae\_cheol/%EB%A9%80%ED%8B%B0%EB%B8%8C%EB%9E%9C%EC%B9%98-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8](https://velog.io/@jae\_cheol/%EB%A9%80%ED%8B%B0%EB%B8%8C%EB%9E%9C%EC%B9%98-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8)
