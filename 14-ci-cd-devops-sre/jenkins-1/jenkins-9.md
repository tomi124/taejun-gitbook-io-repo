# Jenkins 9 - 블루오션



### 젠킨스 블루오션 <a href="#undefined" id="undefined"></a>

* 젠킨스의 주요 애플리케이션에 대한 UI 보조 기능
  * 향상된 시각화
  * 파이프라인 에디터
  * 개인화
  * 깃과 깃허브를 위한 쉽고 빠른 파이프라인 설정 마법사

### 설치 <a href="#undefined" id="undefined"></a>

* Jenkins 관리 > 플러그인 관리 > 설치가능(Available) > Blue Ocean검색 후 설치
* 설치가 끝나면 대시보드에 블루오션이 생긴것을 볼 수 있다.\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F57259230-62b3-41e8-8f2f-f8c640e9278b%2Fimage.png)

### 블루오션 살펴보기 <a href="#undefined" id="undefined"></a>

1. 블루오션 대시보드\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fecdd33d9-10e8-4633-925f-2c190e00b780%2Fimage.png)
2. 멀티브랜치 파이프라인\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F5cedf25c-46fe-4935-bb9e-ee7dda56de74%2Fimage.png)

### 블루오션에서 파이프라인 생성 <a href="#undefined" id="undefined"></a>

* 블루오션 대시보드에서 새로운 파이프라인 버튼을 클릭하고 단계에 따라 진행하면 된다.\
  아래 이미지에서는 토큰 입력이 없지만 있다면 토큰 생성 링크 클릭 후 생성하면 된다.\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F013f7300-9c90-4ef2-accb-9b105f38f80a%2Fimage.png)\
  위 이미지처럼 선택 후 파이프라인 생성 클릭하면 다음 이미지와 같다.\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fb64f2280-4a52-40cb-a5a7-dbad92df398a%2Fimage.png)\
  에이전트 부분을 다음과 같이 선택하고 입력한다.\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F0bc8c6f9-5424-44a1-bbd4-a633c5460696%2Fimage.png)\
  다음으로 가운데 + 버튼을 클릭하면 Build Stage를 만들 수 있다. 스테이지 이름 입력칸에 Build라고 작성 후 스텝 추가 버튼 클릭\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fdcfee143-b22d-42ff-b395-2efa1d9cbc54%2Fimage.png)\
  provide를 검색해서 Provide Maven environment 선택\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fa2af972b-0a91-4dcd-b0d6-85b71362330e%2Fimage.png)\
  Maven 영역에 M3 입력 후 나머지는 그대로 놔둔다.\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F327febea-ea16-472b-8899-63961cf19355%2Fimage.png)\
  스텝 추가 클릭 후 Shell Script 선택 후 mvn clean install 입력\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F594612c4-fe3b-442c-b524-26493ac11fa7%2Fimage.png)\
  그리고 뒤로가기 화살표를 클릭해 이전 메뉴로 돌아가자(아래 이미지는 Build 스탭 다 입력 시 아래와 같음)\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F361df112-6c2c-417b-87f4-09ad09858a31%2Fimage.png)\
  다음 Build 다음의 + 버튼을 클릭하고 Results라고 입력한다\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F4c15c1a4-a84d-49c5-8cf8-2280a46ed397%2Fimage.png)\
  스텝 추가 후 Junit 검색 후 Archive JUnit-formatted test results 선택 하고 TestResults 칸에 \*\*/target/surefire-reports/TEST-_.xml 입력한다._\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fc748346d-1ccf-42ba-8527-d52786b25f96%2Fimage.png)\
  _뒤로가기 클릭 후 스텝 추가 클릭 후 archive 검색 > Archive the artifacts선택 > Artifacts 영역에 target/_.jar 작성\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fa716953b-bf68-49e1-ad03-c8e48ac4892d%2Fimage.png)\
  뒤로가기 클릭 후 우측 상단 저장버튼 클릭\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F4f426079-2885-44e8-8ea9-36b025346f33%2Fimage.png)\
  ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F95a73056-0441-4825-ab75-55e86a51a714%2Fimage.png)\
  Save & run 버튼 클릭하면 레포지토리에 Jenkinsfile이 생성된 것을 확인 할 수있다.



출처 : [https://velog.io/@jae\_cheol/%EC%A0%A0%ED%82%A8%EC%8A%A4-%EB%B8%94%EB%A3%A8%EC%98%A4%EC%85%98](https://velog.io/@jae\_cheol/%EC%A0%A0%ED%82%A8%EC%8A%A4-%EB%B8%94%EB%A3%A8%EC%98%A4%EC%85%98)
