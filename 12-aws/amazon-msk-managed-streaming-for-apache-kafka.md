# Amazon MSK(Managed Streaming for Apache Kafka)



## 1. 🙌 Amazon MSK 란 <a href="#1-amazon-msk" id="1-amazon-msk"></a>

Amazon MSK (Managed Streaming for Apache Kafka) 는 Apache Kafka를 사용하여 스트리밍 데이터를 처리하는 애플리케이션을 빌드하고 실행할 수 있는 완전관리형 서비스입니다. Apache Kafka는 실시간 스트리밍 데이터 파이프라인 및 애플리케이션을 구축하기 위한 오픈 소스 플랫폼입니다. Amazon MSK를 통해 네이티브 Apache Kafka API를 사용하여 데이터 레이크를 채우고, 데이터베이스와 변경 사항을 스트리밍 방식으로 주고받으며, 기계 학습 및 분석 애플리케이션을 지원할 수 있습니다.

또한 Amazon MSK는 클러스터 생성, 업데이트 및 삭제 등에 필요한 제어 영역 작업을 제공합니다. 따라서 데이터 생성 및 소비와 같은 Apache Kafka 데이터 영역 작업을 사용할 수 있습니다.

![](https://media.vlpt.us/images/busybean3/post/d54a1466-3ff6-4aa2-8ca8-2156bcaa8128/image.png)

Apache Kafka 클러스터를 프로덕션에서 설정하고 규모를 조정하고 관리하는 것은 까다로운 작업입니다. 혼자서 Apache Kafka를 실행하는 경우 서버 프로비저닝, Apache Kafka 수동 구성, 실패할 경우 서버 교체, 서버 패치 및 업그레이드 오케스트레이션, 고가용성을 위한 클러스터 설계, 데이터가 지속적으로 저장되었으며 안전한지 확인, 모니터링 및 경보 설정, 부하 변경을 지원하기 위한 크기 조정 이벤트의 신중한 계획 등을 모두 직접 수행해야 합니다. Amazon MSK를 사용하면 Apache Kafka 인프라 관리에 대한 전문성이 없이도 Apache Kafka에서 편리하게 프로덕션 애플리케이션을 구축하고 실행할 수 있습니다. 이를 통해 인프라 관리 시간을 줄이고, 더 많은 시간을 애플리케이션 개발에 활용할 수 있습니다.

Amazon MSK 콘솔에서 몇 번만 클릭하면 Apache Kafka의 배포 모범 사례를 기반으로 하는 설정과 구성으로 고가용성 Apache Kafka 클러스터를 만들 수 있습니다. Amazon MSK는 Apache Kafka 클러스터를 자동으로 프로비저닝하고 실행합니다. Amazon MSK는 클러스터 상태를 지속적으로 모니터링하고, 애플리케이션 가동 중지 없이 비정상적인 노드를 자동으로 교체합니다. 또한 Amazon MSK는 미사용 데이터를 암호화하여 Apache Kafka 클러스터를 안전하게 유지합니다.

## 2. 👍 장점 <a href="#2" id="2"></a>

### 2-1. 완벽한 호환성 <a href="#2-1" id="2-1"></a>

> Amazon MSK는 Apache Kafka를 자동으로 실행하고 관리합니다. 따라서 애플리케이션 코드를 변경하지 않고도 손쉽게 기존 Apache Kafka 애플리케이션을 AWS로 마이그레이션하여 실행할 수 있습니다. Amazon MSK를 사용하면 오픈 소스 호환성을 유지하고, MirrorMaker, Apache Flink 및 Prometheus와 같은 친숙한 사용자 지정 및 커뮤니티 구축 도구를 계속 사용할 수 있습니다.

### 2-2. 완전관리형 <a href="#2-2" id="2-2"></a>

> Amazon MSK를 사용하면 Apache Kafka 환경 관리의 운영 오버헤드에 대한 걱정 없이 스트리밍 애플리케이션 생성에 집중할 수 있습니다. Amazon MSK에서 자동으로 Apache Kafka 클러스터와 Apache ZooKeeper 노드의 프로비저닝, 구성 및 유지를 관리합니다. 또한 Amazon MSK는 AWS 콘솔에 주요 Apache Kafka 성능 지표를 표시합니다.

### 2-3. 탄력적인 스트림 처리 <a href="#2-3" id="2-3"></a>

> Apache Flink는 스트리밍 데이터의 상태 저장 계산을 위한 강력한 오픈 소스 스트림 처리 프레임워크입니다. SQL, Java 또는 Scala로 작성된 완전관리형 Apache Flink 애플리케이션을 실행하여 Amazon MSK 내에서 데이터 스트림을 처리하도록 탄력적으로 확장할 수 있습니다.

### 2-4. 뛰어난 가용성 <a href="#2-4" id="2-4"></a>

> Amazon MSK는 Apache Kafka 클러스터를 생성하고 AWS 리전 내에서 다중 AZ 복제를 제공합니다. Amazon MSK는 클러스터 상태를 지속적으로 모니터링하고, 구성 요소가 실패할 경우 자동으로 대체합니다.

### 2-5. 뛰어난 보안 <a href="#2-5" id="2-5"></a>

> Amazon MSK는 VPC 네트워크 격리, 컨트롤 플레인 API 권한 부여를 위한 AWS IAM, 저장 데이터 암호화, 전송 데이터 TLS 암호화, TLS 기반 인증서 인증, AWS Secrets Manager를 통해 보호되는 SASL/SCRAM 인증, 데이터 플레인 권한 부여를 위한 Apache Kafka ACL(액세스 제어 목록) 지원을 비롯하여 Apache Kafka 클러스터를 위한 다양한 수준의 보안을 제공합니다.

## 3.🤝 작동 방식 <a href="#3" id="3"></a>

다음 다이어그램은 Amazon MSK의 작동 방식에 대한 개요입니다.

![](https://media.vlpt.us/images/busybean3/post/2e5b63a1-d08a-422a-b50c-76182c9aeeed/image.png)

다이어그램은 다음 구성 요소 간의 상호 작용을 보여 줍니다.

> * Broker 노드— Amazon MSK 클러스터를 생성할 때 Amazon MSK가 각 가용 영역에 생성할 브로커 노드 수를 지정합니다. 이 다이어그램에 표시된 예제 클러스터에는 가용 영역당 하나의 브로커가 있습니다. 각 가용 영역에는 고유한 virtual private cloud(VPC) 서브넷이 있습니다.
> * ZooKeeper 노드— Amazon MSK는 Apache ZooKeeper 노드도 만듭니다. Apache ZooKeeper는 안정성이 뛰어난 분산 조정을 지원하는 오픈 소스 서버입니다.
> * Producer, Consumer 및 Topic 제작자— Amazon MSK를 사용하면 Apache Kafka 데이터 영역 작업을 사용하여 Topic을 만들고, 데이터를 생산하고 소비할 수 있습니다.
> * Cluster 작업 - 다음을 수행할 수 있습니다.AWS Management Console,AWS Command Line Interface(AWS CLI) 또는 SDK의 API를 사용하여 제어 영역 작업을 수행할 수 있습니다. 예를 들어 Amazon MSK 클러스터를 만들거나 삭제하고, 계정의 모든 클러스터를 나열하고, 클러스터의 속성을 보고, 클러스터의 브로커 수와 유형을 업데이트할 수 있습니다.

Amazon MSK는 클러스터에 대한 가장 일반적인 결함 시나리오를 감지하고 자동으로 복구하므로 생산자와 소비자 애플리케이션이 최소한의 영향을 받으면서 쓰기 및 읽기 작업을 계속할 수 있습니다. Amazon MSK가 브로커 결함을 감지하면 해당 결함을 완화하거나 비정상적 또는 연결할 수 없는 브로커를 새 브로커로 바꿉니다. 또한 가능한 경우 이전 브로커의 스토리지를 재사용하여 Apache Kafka가 복제해야 하는 데이터를 줄입니다. 가용성에 미치는 영향은 Amazon MSK가 감지 및 복구를 완료하는 데 필요한 시간으로 제한됩니다. 복구 후 생산자와 소비자 앱은 결함 이전에 사용한 것과 동일한 브로커 IP 주소와 계속 통신할 수 있습니다.

## 4.👊 기능 <a href="#4" id="4"></a>

### 4-1. 오픈 소스 <a href="#4-1" id="4-1"></a>

*   네이티브 Apache Kafka를 통해 실행

    > Amazon MSK는 네이티브 버전의 Apache Kafka 배포를 지원합니다. 따라서 Apache Kafka용으로 구축된 애플리케이션 및 도구를 애플리케이션 코드 변경 없이 Amazon MSK에서 즉시 사용할 수 있습니다.
*   간소화된 버전 가용성

    > Amazon MSK는 일반적으로 Apache Kafka의 최신 버전을 공개된 날짜로부터 7일 내에 제공합니다.
*   원할한 버전 업그레이드

    > Amazon MSK 클러스터의 Apache Kafka 버전을 몇 번의 클릭만으로 업그레이드하여 새로운 Apache Kafka 버전에 제공되는 기능 및 버그 수정을 활용할 시기를 결정할 수 있습니다. Amazon MSK는 실행 중인 클러스터의 버전 업그레이드 배포를 자동화하여 모범 사례에 따라 고객의 클라이언트 I/O 가용성을 유지합니다.

### 4-2. 관리할 서버 없음 <a href="#4-2" id="4-2"></a>

*   완전관리형

    > 콘솔에서 클릭 몇 번으로 Apache Kafka의 배포 모범 사례를 준수하는 완전관리형 Apache Kafka 클러스터를 개발할 수 있습니다. 사용자 지정 구성을 활용하여 자체 클러스터를 개발할 수도 있습니다. 사용자가 원하는 구성을 생성하면 Amazon MSK에서는 사용자 Apache Kafka 클러스터 및 Apache ZooKeeper 노드의 작업을 자동으로 프로비저닝하고, 구성하고, 관리합니다.
*   Apache ZooKeeper 포함

    > Apache Kafka를 실행하고 클러스터 작업을 조율하며 클러스터와 상호 작용하는 리소스의 상태를 유지하려면 Apache ZooKeeper가 필요합니다. Amazon MSK는 사용자를 대신하여 Apache ZooKeeper 노드를 관리합니다. 각 Amazon MSK 클러스터에는 Apache Kafka 클러스터를 위한 적절한 수의 Apache ZooKeeper 노드가 무료로 포함되어 있습니다.

### 4-3. 뛰어난 가용성 <a href="#4-3" id="4-3"></a>

*   고가용성이 기본값

    > 모든 클러스터가 여러 가용 영역(AZ 3개가 기본값)에 프로비저닝되고 Amazon MSK의 서비스 수준 계약으로 지원되며 클러스터 인프라 및 Apache Kafka 소프트웨어 내의 문제를 탐지하고 대응하는 자동화된 시스템으로 지원됩니다. 구성 요소 장애가 발생하면 애플리케이션 가동 중단 시간 없이 Amazon MSK가 자동으로 구성 요소를 교체합니다. Amazon MSK는 Apache ZooKeeper 노드의 가용성을 관리하므로 사용자가 노드를 시작, 중지 또는 직접 액세스할 필요가 없습니다. 또한 Amazon MSK는 필요에 따라 소프트웨어 패치를 자동으로 배포하여 클러스터를 최신 상태로 유지하고 원활한 실행을 보장합니다.
*   데이터 복제

    > Amazon MSK는 고가용성을 위해 다중 AZ 복제를 활용합니다. 데이터 복제는 추가 비용 없이 제공됩니다.

### 4-4. 뛰어난 보안 <a href="#4-4" id="4-4"></a>

*   비공개 연결

    > Apache Kafka는 Amazon MSK가 관리하는 Amazon VPC에서 실행됩니다. 클러스터는 사용자가 지정한 구성에 따라 Amazon VPC, 서브넷 및 보안 그룹에서 사용할 수 있습니다. 네트워크 구성을 완벽하게 통제할 수 있으며, VPC의 IP 주소는 ENI(탄력적 네트워크 인터페이스)를 통해 Amazon MSK 리소스로 연결됩니다.
*   세분화된 액세스 제어

    > IAM 액세스 제어는 무료 보안 옵션이며, IAM 역할 또는 사용자 정책을 사용하여 액세스를 제어하는 클러스터 인증 및 Apache Kafka API 인증을 단순화합니다. IAM 액세스 제어를 사용하면 고객은 Apache Kafka에 대한 클라이언트 인증 및 승인을 제어하기 위해 더 이상 일회성 액세스 관리 시스템을 구축하여 실행할 필요가 없으며 클러스터는 기본적으로 최소한의 권한을 사용하여 보호됩니다. SASL/SCRAM 또는 상호 TLS 인증을 Apache Kafka 액세스 제어 목록(ACL)과 함께 사용하여 클라이언트 액세스를 제어할 수도 있습니다.
*   암호화

    > Amazon MSK는 특수 구성 또는 서드 파티 도구를 사용하지 않고 저장 중 데이터를 암호화합니다. 모든 저장 중 데이터는 기본적으로 AWS Key Management Service(KMS) CMK(고객 마스터 키)를 사용하거나 고유한 CMK를 사용하여 암호화될 수 있습니다. Amazon MSK는 브로커와 클라이언트 사이, 그리고 클러스터에서 브로커와 클라이언트 사이에서 TLS를 통한 전송 데이터도 암호화합니다.

### 4-5. 최저 비용 <a href="#4-5" id="4-5"></a>

> Amazon MSK를 통해 일일 2.50 USD 미만의 요금으로 시작할 수 있습니다. 고객은 일반적으로 모든 비용을 포함하여 수집된 GB당 0.05 USD 및 0.07 USD를 지불합니다. 이는 다른 관리형 공급자 요금의 1/13에 해당합니다. 현재 요금을 확인하려면 Amazon MSK 요금 페이지로 이동하고 Amazon MSK 클러스터의 크기를 적절하게 조정하는 방법에 대해 알아보려면 Amazon MSK 모범 사례 페이지로 이동하세요.

![](https://media.vlpt.us/images/busybean3/post/5a01655a-98fa-4ebf-af32-0ce678e591da/image.png)

### 4-6. 긴밀하게 통합 <a href="#4-6" id="4-6"></a>

> Amazon MSK에서 제공되는 AWS 통합의 범위와 깊이를 따라올 수 있는 공급자는 없습니다. 이러한 통합에는 다음이 포함됩니다.
>
> > * Apache Kafka 및 서비스 수준 API 액세스 제어를 위한 AWS IAM
> > * 완전관리형 Apache Flink 애플리케이션을 실행하여 Apache Kafka 내의 스트리밍 데이터를 처리하기 위한 Amazon Kinesis Data Analytics
> > * 스키마를 중앙에서 제어하고 변경하는 AWS Glue 스키마 레지스트리
> > * IoT 이벤트를 MSK로 스트리밍하는 AWS IoT
> > * 변경 데이터 캡처 및 분석을 위한 AWS DMS
> > * 프라이빗 클라이언트 연결 및 네트워크 격리를 위한 AWS VPC
> > * 유휴 시 암호화를 위한 AWS KMS
> > * 상호 TLS 클라이언트 인증을 위한 AWS Certificate Manager 프라이빗 CA
> > * SASL/SCRAM 보안 정보의 안전한 저장 및 관리를 위한 AWS Secrets Manager
> > * Amazon MSK를 코드로 배포하는 AWS CloudFormation
> > * 클러스터, 브로커, 주제, 소비자 및 파티션 수준 지표를 위한 Amazon CloudWatch

### 4-7. 확장성 <a href="#4-7" id="4-7"></a>

*   브로커 규모 조정

    > Apache Kafka 브로커의 크기 또는 패밀리를 변경하여 가동 중단 없이 몇 분 안에 Amazon 클러스터 규모를 조정할 수 있습니다. 브로커 크기 또는 패밀리를 변경하는 방법은 Amazon MSK 클러스터 규모를 조정할 때 가장 많이 사용되는 방법입니다. 워크로드의 변화에 맞춰 MSK 클러스터의 컴퓨팅 용량을 유연하게 조정할 수 있기 때문입니다. 이 방법은 Apache Kafka 가용성에 영향을 줄 수 있는 파티션 재할당이 필요하지 않다는 점에서 선호됩니다.\
    > 콘솔 또는 명령줄 인터페이스(CLI)를 사용하여 클러스터당 최대 100개의 브로커까지 클러스터 크기를 확장할 수도 있습니다. 클러스터당 16개 이상의 브로커 또는 계정당 31개 이상의 브로커가 필요한 경우 한도 증가 요청을 제출하세요.
*   자동 파티션 관리

    > Amazon MSK는 파티션 할당을 자동으로 관리하는 Apache Kafka용 오픈 소스 도구인 Cruise Control과 통합됩니다.
*   스토리지 규모 자동 조정

    > AWS Management Console이나 AWS CLI를 사용하여 스토리지 요구 사항이 변경되면 그에 맞게 브로커별로 프로비저닝되는 스토리지 용량을 매끄럽게 확장할 수 있습니다. 또는, Auto Scaling 정책을 만들어 스트리밍 요구 사항에 맞게 스토리지를 자동 확장할 수 있습니다.

### 4-8. 구성 가능 <a href="#4-8" id="4-8"></a>

> Amazon MSK는 기본적으로 Apache Kafka에 대한 모범 사례 클러스터 구성을 배포하고, 모든 동적 및 주제 수준 구성을 지원하는 동시에 30개 이상의 서로 다른 클러스터 구성을 튜닝할 수 있는 기능을 고객에게 제공합니다. 자세한 내용은 설명서에서 사용자 지정 MSK 구성을 참조하세요.

### 4-9. 시각화 <a href="#4-9" id="4-9"></a>

*   기본 CloudWatch 지표

    > Amazon CloudWatch를 사용하여 중요한 클러스터, 브로커, 주제, 소비자 및 파티션 수준 지표를 시각화하고 모니터링할 수 있습니다.
*   JMX 및 Node 지표를 Prometheus 서버로 내보내기

    > Prometheus의 오픈 모니터링을 사용하면 Datadog, Lenses, New Relic, Sumo logic 또는 Prometheus 서버와 같은 솔루션을 사용하여 Amazon MSK를 모니터링하고 기존 모니터링 대시보드를 Amazon MSK로 손쉽게 마이그레이션할 수 있습니다. 자세한 내용은 설명서에서 Prometheus를 사용한 오픈 모니터링을 참조하세요.

## \[REFERENCE] <a href="#reference" id="reference"></a>

* [https://velog.io/@busybean3/Amazon-MSKManaged-Streaming-for-Apache-Kafka-%EB%9E%80](https://velog.io/@busybean3/Amazon-MSKManaged-Streaming-for-Apache-Kafka-%EB%9E%80)
* [https://docs.aws.amazon.com/ko\_kr/msk/latest/developerguide/what-is-msk.html](https://docs.aws.amazon.com/ko\_kr/msk/latest/developerguide/what-is-msk.html)
* [https://aws.amazon.com/ko/msk/](https://aws.amazon.com/ko/msk/)
