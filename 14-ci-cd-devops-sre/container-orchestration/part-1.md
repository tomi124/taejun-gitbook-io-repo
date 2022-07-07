# 쿠버네티스 기초 Part 1



* [https://www.slideshare.net/pyrasis/docker-fordummies-44424016](https://www.slideshare.net/pyrasis/docker-fordummies-44424016)



### Container Orchestration

#### 컨테이너 오케스트레이션이란?

**단순 배포 관리**

**컨테이너와 호스트의 증가 -> 컨테이너 배포 프로세스의 최적화 필요**

결국 더 많은 최적화 방법들이 논의되었고 자동화가 필요해졌으며 이러한 최적화 행위(기능)를 컨테이너 오케스트레이션이라고 합니다.

**오케스트레이션 기능**

* 컨테이너 자동 배치 및 복제
* 컨테이너 그룹에 대한 로드 밸런싱
* 컨테이너 장애 복구
* 클러스터 외부에 서비스 노출
* 컨테이너 추가 또는 제거로 확장 및 축소
* 컨테이너 서비스간의 인터페이스를 통한 연결 및 네트워크 포트 노출 제어

### 컨테이너 오케스트레이터

**컨테이너 오케스트레이터들은 특정 그룹의 호스트틀을 클러스터라는 형태로 묶어서 위와 같은 요구사항들을 충족시킬 수 있도록 도와주는 도구들입니다.**

현재 사용할 수 있는 많은 컨테이너 오케스트레이터들이 있습니다.

**Docker Swarm**

```
Docker.Inc에서 제공하며 Docker Engine에 포함되어있습니다. 
```

**쿠버네티스**

```
구글에서 시작되었지만 현재는 클라우드 네이티브 파운데이션이라는 프로젝트의 한 부분에 포함되었습니다. ( de facto standard ) 
```

**아마존 ECS**

```
AWS에서 그들의 인프라스트럭쳐 위에 도커 컨테이너들을 실행 시킬 수 있도록 제공하는 서비스
```

**그외 여러가지 ( Mesos Marathon ,Hashicorp Nomad )**

### 컨테이너 오케스트레이터를 왜 사용하나?

**일정 규모의 컨테이너들은 수동으로 혹은 스크립트들을 사용해서 운영할 수 있다고 논쟁을 할수는 있겠지만 컨테이너 오케스트레이터들은 그러한 작업을 더 쉽게 만들 수 있습니다.**

* 여러 호스트들을 모아서 클러스터의 일부로 만들 수 있습니다.
* 컨테이너들을 서로 다른 호스트에서 동작하도록 스케줄링할 수 있습니다.
* 한 호스트에 동작하는 컨테이너가 다른 호스트에 동작하는 컨테이너와 연결할 수 있습니다.
* 컨테이너들과 저장공간을 묶을 수도 있습니다.
* 비슷한 타입의 컨테이너들을 상위 레벨 개념( 예를들면 서비스)과 연결할 수 있어서 개별 컨테이너들을 관리할 필요가 없습니다.
* 리소스 사용량에 대한 체크를 할 수 있어 필요할 때 리소스를 최적화할 수 있습니다.
* 컨테이너 안에서 동작하는 어플맄이션들에 보안에 안전한 접근을 제공할 수 있습니다.

이러한 모든 기본 제공 이점을 통해 컨테이너를 관리하는 데 컨테이너 오케스트레이터를 사용하여 관리하는 게 좋습니다.\
쿠버네티스를 공부하는 이유가 되겠죠

### Kubernetes

#### 쿠버네티스 웹사이트에서는 아래와 같이 정의하고 있습니다.

```
"Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications."   
```

#### 알쓸신잡

**쿠버네티스는 그리스 어 κυβερνήτης (kyvernitis) 키잡이, 조타수라는 뜻으로 컨테이너 선의 관리자 정도로 생각해볼 수 있습니다.**

**k8s 라고도 표현하는데 k와s 사이에 8개 문자가 있어서 약자로 사용합니다.**

**쿠버네티스는 Google내에서 십여년 동안 구글 서비스의 컨테이너 관리를 담당했던 Borg System에 영향을 많이 받아 Borg에서 사용하던 기능과 모델들을 차용했습니다.**

```
		API Server / Pod / IP-per-Pod / Services / Labels 등 
```

### 쿠버네티스 기능 Part 1

**Automatic binpacking**

```
가동성에 영향을 주지 않고 사용량과 제약사항에 근거해서 자동으로 컨테이너들을 동작시킵니다. 
```

**Self-healing**

```
문제가 생긴 노드들로부터 컨테이너들의 위치를 조절하거나 재시작을 자동으로 해줍니다. 정책과 룰에 기반한 헬스 체크에 응답하지 않는 컨테이너들을 죽이거나 재시작합니다. 
```

**Horizontal scaling**

```
CPU / 메모리 같은 리소스 사용량에 근거해서 자동으로 어플리케이션들을 확장할 수 있습니다.  
```

**Service discovery and Load balancing**

```
컨테이너들의 집합을 그룹핑할 수 있고 그들을 DNS 를 통해 참조할 수 있습니다. 이 DNS는 쿠버네티스 서비스라고 불리기도 합니다. 
쿠버네티스는 이러한 서비스들을 자동으로 찾을 수 있고 고객의 요청을 주어진 서비스의 컨테이너들간에 부하 분산을 합니다. 
```

### 쿠버네티스 기능 Part 2

**Automated rollouts and rollbacks**

```
어플리케이션 다운 타임 없이 신규 버전이나 설정으로 롤 아웃을 하거나 롤백을 제공합니다. 
```

**Secretes and configuration management**

```
별도의 이미지 재 빌딩없이 기밀 데이터및 설정 정보를 관리합니다. 
```

**Storage orchestration**

```
쿠버네티스 또는 해당 플러그인을 통해 소프트웨어 정의 스토리지를 기반으로 로컬, 외부 및 스토리지 솔루션을 컨테이너에 자동으로 탑재 할 수 있습니다 
```

**Batch Execution**

```
배치 지원 
```

### Kubernetes Architecture - Overview

￼￼![](http://whatsup.nhnent.com/owfs/read/164392/3480fb19-0832-4ab0-b977-f7552c01e1b6)

### Master Node

**마스터 노드는 쿠버네티스 클러스터를 관리하는 역할을 담당하며 모든 관리 업무의 시작점이 됩니다.**

**이 마스터 노드들과는 통해서 CLI, GUI ( dashboard ) , API들을 통해서 커뮤니케이션을 할 수 있습니다.**

**장애 방지를 위해 클러스를 구성할 때 마스터 노드는 한 개 이상을 만들 수 있습니다. 만약 우리가 한개 이상의 마스터 노드들을 가지고 있다면 해당 마스터 노드들은 HA 모드로 동작하고 그들 중의 하나가 모든 오퍼레이션들을 수행하는 리더가 되고 나머지는 팔로워가 됩니다. ( like zookeeper )**

**클러스터 상태를 관리하기 위해 쿠버네티스는 etcd를 사용하며 각 마스터 노드들은 etcd와 연결합니다. etcd는 분산 키-밸류 스토어이며 마스터 노드의 한 부분에서 혹은 물리적으로 외부에 위치하도록 설정할 수 있습니다.**

**하나의 마스터 노드들은 다음과 같은 컴포넌트들을 가지고 있습니다.**

### Master Node Component : API Server

**모든 관리 행위는 API 서버를 통해서 수행됩니다. 한명의 사용자가 Rest 명령을 API 서버에 전송하면 그 요청을 검증하고 실행합니다.**

**해당 요청을 수행한 이후에 클러스터의 수행 상태는 etcd 에 저장합니다.**

### Master Node Components: Controller Manager

**컨트롤러 매니저는 클러스터의 상태를 통제하는 컨트롤 루프들을 관리합니다.**

**각각의 컨트롤 루프들은 관리하는 오브젝트(Pod, Service, ReplicaSet 등)들의 기 정의된 정상 상태를 알고 있고 그들의 현재 상태를 API 서버를 통해서 모니터링 합니다.**

**컨트롤 루프 안에서 현재 오브젝트의 상태가 원하는 상태에 맞지 않으면 원하는 상태와 맞게끔 적절한 동작을 수행합니다.**

### Master Node Components: Scheduler

**이름에서도 유추할 수 있듯이 서로 다른 워커 노드들의 업무를 스케줄링합니다**

**해당 스케줄러는 워커 노드들의 리소스 사용량 정보를 알 수 있고 사용자가 지정한 제약사항( 예를 들면. label==ssd 를 가진 노드의 작업 관리 )에 대해서도 알고 있습니다.**

### Master Node Components: etcd

**etcd는 클러스터의 상태를 저장하는 분산된 key-value 스토어입니다.**

**쿠버네티스 마스터의 한 부분이 될 수도 있고 독립적으로 외부에 설치될 수도 있으며 이럴 경우는 마스터 노드들이 외부의 세팅된 지점에 연결합니다.**

**Raft Consensus Algorithm 을 사용한 분산 저장소 ( like Zookeeper )**

**Go 언어로 작성되어있으며 cluster 상태 정보 외에 subnets, ConfigMaps, Secrets 등과 같은 설정 상세정보를 저장하는데도 사용됩니다.**

### Worker Node

**워커노드는 하나의 머신으로 ( VM이나 물리서버 등등 ) Pods를 이용해서 어플리케이션을 실행시키며 마스터노드에 의해 관리됩니다.**

**Pod 는 워커노드 위에서 동작하도록 스케줄되며 워커노드는 Pod와 연결하고 실행시키기 위한 툴들을 가지고 있습니다.**

**하나의 Pod는 쿠버네티스에서 스케쥴링하는 하나의 단위입니다. 스케쥴링 단위라는 것은 한개 혹은 그 이상의 컨테이너들의 논리적 컬렉션이며 언제나 함께 스케줄링 됩니다.**

**또한 외부 세상에서 어플리케이션에 접근하기 위해서 마스터 노드가 아닌 워커 노드들에 연결합니다.**

**워커 노드들은 다음과 같은 컴포넌트들을 가지고 있습니다.**

### Worker Node Components: Container Runtime

**컨테이너를 실행시키고 해당 컨테이너의 라이프사이클을 관리하기 위해서 워커노드에 컨테이너 런타임이 필요합니다.**

**컨테이너 런타임에는 다음과 같은 것들이 있습니다.**

* **containerd**
  * docker = containerd를 사용하는 플랫폼
* **rkt**
  * core OS 제공. container runtime - rocket 이라고 읽음
* **lxd**
  * lightweight container hypervisor

### Worker Node Components: kubelet

**kubelet은 마스터 노드와 커뮤니케이션을 담당하는 워커 노드에서 동작하는 agent 입니다.**

**API 서버를 통해서 Pod 정의를 수신하며 해당 Pod와 연관있는 컨테이너들을 실행합니다.**

**Pod들의 부분인 컨테이너들이 정상동작(Liveness Prove) 중인지를 확인합니다.**

**kubelet은 컨테이너 런타임 인터페이스를 통해서 컨테이너 런타임에 연결합니다.**

### Worker Node Components: kube-proxy

**어플리케이션에 액세스하기 위해서 Pod들에 직접 연결하는 대신 연결 포인트로 서비스라고 불리는 논리적 구조체를 사용합니다.**

**하나의 서비스는 연관된 Pod들을 그룹핑하고 로드밸런싱 합니다.**

**kube-Proxy는 각 워커 노드들에서 동작하는 네트워크 프록시로서 쿠버네티스 API를 통해 정의한 서비스에 클라이언트가 연결할 수 있도록 하는 것이 목적입니다.**

### 실습 환경 ( MiniKube 설치 )

#### 사전 준비 사항

기본 개발 환경이  Mac인걸 가정해서 작성된 실습사항입니다. Windows 사용자를 위한 부분은 생략합니다.&#x20;

#### 본인 컴퓨터에 Docker for mac 설치 ( 본인 PC에 Docker 환경 구축 )&#x20;

#### [https://docs.docker.com/docker-for-mac/install/](https://docs.docker.com/docker-for-mac/install/)

**Virtual Box 설치**

* MiniKube는 VM 환경에서 동작하게 되어있습니다. 적절한 아래 링크에서 VM 환경을 다운로드 하시면 됩니다.&#x20;
* [https://kubernetes.io/docs/tasks/tools/install-minikube/#before-you-begin](https://kubernetes.io/docs/tasks/tools/install-minikube/#before-you-begin)

**HomeBrew를 통한 minikube 설치**

* brew cask install minikube

**에러가 발생하는 경우 기존에 kubectl 이 설치되어 있어서 링크가 깨진 경우임.**

* link overwrite 해서 사용\
  ![](http://whatsup.nhnent.com/owfs/read/164393/e0497ec5-1e93-4af2-ae09-d5ca1a015aa9)\


**혹은 직접 다운로드 후 설치**

**curl -Lo minikube** [**https://storage.googleapis.com/minikube/releases/v0.25.0/minikube-darwin-amd64**](https://storage.googleapis.com/minikube/releases/v0.25.0/minikube-darwin-amd64) **&& chmod +x minikube && sudo mv minikube /usr/local/bin/**

**$ minikube start**

**$ minikube status**

**$ minikube dashboard**&#x20;

아래 화면이 뜨는지만 확인하시면 됩니다. (  쿠버네티스 기본 GUI 환경입니다. ) 아래 내용에 대해서는 다음에 설명합니다.&#x20;

![](http://whatsup.nhnent.com/owfs/read/164394/b9ebf9f4-df3c-421d-9e45-a10fa64cd2ad)\


$ kubectl cluster-info

정상 설치 여부 확인 ( 아래와 같이 클러스터 환경이 구축되었는지 체크합니다. )&#x20;

![](http://whatsup.nhnent.com/owfs/read/164396/95e6cb96-ccc8-4ee0-899f-45bbf316e281)
