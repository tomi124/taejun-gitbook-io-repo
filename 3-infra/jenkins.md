---
description: Jenkins에 대해 알아보자
---

# JENKINS

![https://medium.com/faun/how-to-install-jenkins-on-ubuntu-18-37f4daad1014](<../.gitbook/assets/image (5).png>)

## CI/CD란 무엇인가?

### CI(Continuous Integration)

* 지속적인 통합 : 지속적으로 코드에 대한 통합을 진행함으로써 코드 품질을 유지
* 빌드 및 테스트(유닛테스트 및 통합 테스트)를 통해서 병합 진행
* 통합이 완료된 최신 버전을 제공

### CD(Continuous Delivery)

* 지속적인 배포 : 통합과정을 통과한 버전을 사용자 설정에 따라 주기적으로 배포
* 프로덕션 환경으로 배포할 준비가 되어 있는 코드베이스를 확보하는 것이 목표

### CD(Continuous Deployment)

* CI/CD 파이프라인의 마지막 단계
* 배포될 서버나 배포 전략을 코드로 관리하여 형상관리 가능
* 분산 환경에 배포할 경우 더욱 중요하다

## Jenkins

* 소프트웨어 개발 시 지속적으로 통합 서비스를 제공하는 툴 – CI (Continuous Integration)
* 젠킨스와 같은 CI 툴이 등장하기 전에는 일정시간 마다 빌드를 실행하는 방식으로 진행 ( 보통 모든 커밋이 끝난 심야 시간대에 이러한 빌드가 집중적으로 진행 – nightly-build )

### Jenkins를 왜 사용할까?

* 프로젝트 표준 컴파일 환경에서의 컴파일 오류 검출
* 자동화 테스트 수행
* 다양한 플러그인 제공 (TFS, GITHUB, GITLAB)
* 간편한 설정 및 사용법



## 참고

* [https://devuna.tistory.com/56](https://devuna.tistory.com/56)
* [https://onlywis.tistory.com/9](https://onlywis.tistory.com/9)
* [https://velog.io/@ash3767/CICD](https://velog.io/@ash3767/CICD)
