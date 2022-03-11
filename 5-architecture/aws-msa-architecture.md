# AWS MSA ARCHITECTURE

![](https://media.vlpt.us/images/rssungjae/post/af4572ef-574f-4d1a-bb36-2b87b6578ce2/994183465B8491190D.png)

[마이크로서비스 아키텍처(MSA)](https://velog.io/@rssungjae/%ED%9B%84%EA%B8%B0%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98MSA#%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98\(msa\))\
[AWS 기반 마이크로서비스 아키텍처 패턴\
](https://velog.io/@rssungjae/%ED%9B%84%EA%B8%B0%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98MSA#aws-%EA%B8%B0%EB%B0%98-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%ED%8C%A8%ED%84%B4)[AWS 기반 마이크로서비스 아키텍처 모범 사례](https://velog.io/@rssungjae/%ED%9B%84%EA%B8%B0%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98MSA#aws-%EA%B8%B0%EB%B0%98-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EB%AA%A8%EB%B2%94-%EC%82%AC%EB%A1%80)\
[소감](https://velog.io/@rssungjae/%ED%9B%84%EA%B8%B0%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98MSA#%EC%86%8C%EA%B0%90)

***

### 마이크로서비스 아키텍처(MSA) <a href="#msa" id="msa"></a>

* MSA란?\
  MicroService Architecture\
  작고, 독립적으로 배포 가능한 각각의 기능을 수행하는 서비스단위, 기능단위로 구성된 프레임워크, 아키텍처
* MSA특징\
  각각 자체 프로세스에서 실행\
  HTTP기반 API로 통신\
  자동화된 배포머신을 통해 독립배포\
  중앙 집중식 관리는 최소화\
  서로 다른 프로그래밍 언어로 개발 가능\
  서로 다른 데이터 저장 기술을 사용 가능
* MSA장점\
  서비스 별 개별 배포가 가능하다.\
  배포시의 전체서비스의 중단이 없다.\
  특정 서비스, 기능의 확장성이 용이하다.\
  장애가 발생할 경우, 전체 서비스에 영향을 끼치는 부분이 적다.
* MSA단점\
  기능단위의 시스템이 많아져, 인터페이스의 수가 늘어나게되어 오류 발생확인이 어렵다.\
  서비스가 분리되어 있기 때문에 테스트와 트랜잭션이 복잡하다.\
  데이터가 여러 서비스에 분산되어있기때문에 관리가 어렵다.
* 클라우드 시스템인 Azure는 개발 관리도구인 Azure DevOps Services나 Azure Portal등을 이용해서 이러한 단점을 극복하려한다.

![](https://media.vlpt.us/images/rssungjae/post/1fb54886-e75d-4393-b831-8b8b8d6a2c3b/image.png)

***

#### AWS 기반 마이크로서비스 아키텍처 패턴 <a href="#aws" id="aws"></a>

* 기존 Monolith앱의 단점
  * 앱 확장성의 어려움
  * 장기 개발 사이클의 어려움
  * 특정 모듈 장애시의 운영 어려움, 신규 기능 추가의 어려움
* 다양한 마이크로 서비스 구조 진화

![](https://media.vlpt.us/images/rssungjae/post/320a1a2c-7c3b-404e-b7b7-0421aed82ed2/image.png)\
![](https://media.vlpt.us/images/rssungjae/post/404a6cf5-32d4-4447-b6cc-8e444da2c7eb/image.png)

* 마이크로서비스 아키텍처는 자유롭게 만들 수 있다.\
  EC2를 이용해서 서버에 올린다거나, ECS container를 이용해 Docker처럼 올리거나, Lambda를 이용해서 AWS의 다양한 서비스API를 사용하거나, LambdaBlueprints를 이용해서 다양한 외부 서비스 API를 이용해서 개발할 수 있다.

***

#### AWS 기반 마이크로서비스 아키텍처 모범 사례 <a href="#aws" id="aws"></a>

**API 기반 서비스 인터페이스 및 하위 호환성 유지**

lambda버전, API gateway의 stage기능, stages변수를 조합하여 API의 버전을 다르게 배포할 수 있다.(테스트용 등)

**각기 다른 폴리그랏 개발 및 운영 환경**

워크로드별 최적의 개발 환경구성\
Polyglot접근 방식을 통한 서비스 아키텍처 선택

* 적합한 개발 환경, 개발 언어를 사용하여 개발 할 수 있다.\
  개별적인 데이터 스토어를 구성하여, DB트랜잭션 문제가 생겼을 경우, 해당 데이터 스토어만 에러가 나기때문에 그 외기능은 사용가능하다.
* 트랜잭션 에러시에
  * CorrelationID 활용, 환경변수값을 넘겨서 분기별로 확인
  * 서비스별 자체 롤백 제공
* StepFunctions\
  각 단계를 트리거 및 추적, 시각적 워크플로로 확인할 수 있다.

**지속적인 마이크로 서비스 모니터링**

* API 외부적인 요소 모니터링\
  Latency, RPS, Error rate
* API 내부적인 요소 모니터링\
  CloudWatch, OS, Application
* 실시간 로그 대시보드 구성\
  CloudWatch로그를 Elasticsearch로
* 데이터 리포팅 및 시각화\
  S3에 있는 데이터를 Redshift에 넣어 QuickSight, 데이터를 가지고 BI처리화 하여 시각화
* Athena! S3에서 바로 데이터쿼리를 이용하여 데이터 리포팅가능!
* AWS X-Ray\
  완전 관리형 분산 애플리케이션 성능 추적 서비스\
  EC2에 설치하여 사용, 전체 디버깅 및 추적이 가능

**마이크로 서비스별 보안 강화**

* 서비스 수준별 방어
  * 네트워크 레벨\
    VPC, Security Groups, TLS
  * 서버/콘테이너/앱 레벨\
    WAF, Shield
* API Gateway 방어
  * API Throttling
  * API 키 관리

**API 생태계 확산을 통한 비지니스 창출**

* API Gateway Developer Portal\
  외부로 API키 발급및 관리 백엔드 배포
* API Monetization in MarketPlace\
  API 서비스 판매 및 수익화 가능

**기술 너머 조직의 변화**

기능별 조직에서 자율성 높은 책임 조직으로

**자동화**

AWS의 각종 DevOps자동화 도구

***

***

**참조**

* [https://www.youtube.com/watch?v=\_8ylg-DLnBY](https://www.youtube.com/watch?v=\_8ylg-DLnBY)\
  윤석찬 테크 에반젤리스트(AWS 코리아) 강연 참조
