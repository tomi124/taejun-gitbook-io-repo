# Jenkins 6 - 파이프라인 잡 만들기



1. New Item 클릭
2. 다음과 같이 파이프라인 입력후 OK 클릭\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F825a0a03-9669-46a1-9d04-04068c2a5550%2Fimage.png)
3. 파이프라인 탭을 클릭 후 스크립투 부분에 try sample pipeline이라는 셀랙트 박스에서 Scripted Pipleline 선택 후 저장!!\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2Fa6505f6b-79a2-489e-8902-d8cfeaec2ff2%2Fimage.png)
   * 간단히 스크립트\
     node {} 영역 : 메인 컨테이너로 젠킨스 마스터에서 파이프라인 스크립트 영역 전체를 실행하라는 의미\
     stage('Preparation') : 파이프라인을 실행하기 전에 이 단계를 수행해야 한다.\
     stage('Build') : 프로젝트를 빌드함.\
     stage('Results') : 빌드 결과물을 Junit테스트 결과와 함께 묶어낸다.
4. 파이프라인을 실행하기 전에 전역 도구 환결 설정에서 파이프라인에서 사용될 도구를 설정해야함 (ex) 깃, 메이블, 자바, 등등)\
   위치 : Jenkins 관리 -> Global Tool Configuration 로 이동 후 다음과 같이 적용\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F05e31cd0-0a80-4072-8499-ce6203c785bf%2Fimage.png)
5. 젠킨스 대시보드로 가면 다음과 같이 처음에 만든 파이프라인이 리스트에 나온다\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F78e6d24b-1a7a-4709-8072-904625e8d871%2Fimage.png)\
   위의 이미지처럼 빨간 박스부분을 클릭하면 파이프라인이 실행 된다!!
6. 5번의 이미지에서 해당 파이프라인 이름을 클릭하면 해당 파이프라인상세보기로 간다\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F1ac64631-393d-487f-b9b1-65845556a080%2Fimage.png)
7. 빌드한 콘솔을 보기 위해서 빌드 히스토리에 마우스를 올리면 다음과 같이 드롭다운 메뉴가 나타남\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F58e5260f-f58d-47bf-87ba-9ecc941f676b%2Fimage.png)\
   ![](https://velog.velcdn.com/images%2Fjae\_cheol%2Fpost%2F30f22c04-4688-4b13-bc32-3c3055ace013%2Fimage.png)

출처 : [https://velog.io/@jae\_cheol/%EC%A0%A0%ED%82%A8%EC%8A%A4-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EC%9E%A1-%EB%A7%8C%EB%93%A4%EA%B8%B0](https://velog.io/@jae\_cheol/%EC%A0%A0%ED%82%A8%EC%8A%A4-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EC%9E%A1-%EB%A7%8C%EB%93%A4%EA%B8%B0)
