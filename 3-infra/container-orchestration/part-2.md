# 쿠버네티스 기초 Part 2



### 쿠버네티스 첫번째 앱 (도커 복습 )

**Node.js 앱 배포**

먼저 필요한 이미지 내용을 한번 확인해보겠습니다.\
8080 포트를 통해 http 서버를 시작하고 모든 요청에 HTTP 응답 상태 코드 200 OK 와\
텍트스 "You've hit $HostName" 으로 응답하는 간단한 node.js 어플리케이션입니다.

별도의 디렉토리를 하나 만들고 아래 app.js 및 Dockerfile을 하나 만듭니다.

app.js 파일 내용

```
const http = require('http');
const os = require('os');

console.log("Kubia server starting...");

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};

var www = http.createServer(handler);
www.listen(8080);
```

DockerFile

```
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

해당 디렉토리에서 아래 명령어를 통해 이미지를 하나 만듭니다.

```
$ docker build -t kubia . 
```

```
$ docker run --name kubia-container -p 8888:8080 -d kubia 
```

도커를 통해 kubia 이미지에서 kubia-container라는 새 컨테이너를 백그라운드에서 실행하도록 명령입니다.\
이를 통해 http://localhost:8888을 통해 애플리케이션에 액세스 할 수 있습니다.\
![스크린샷 2019-04-28 오후 6.43.16.png](https://nhnent.dooray.com/share/posts/7abiS3cCRWOmBEZOXX32sw/files/2467430259806097095)

#### 이미지 푸시

다른 컴퓨터에서 이미지를 사용하려면 외부 이미지 레지스트리로 푸시해야 하는데 별도의 레지스트리를 구축하지 않았다면 공개 레포지토리 중의 하나인 [도커 허브](http://hub.docker.com)를 사용하겠습니다.

도커 허브는 이미지 스토리지의 이름이 도커 허브 ID로 시작하며 태그부터 지정해야 합니다.

**$docker tag kubia <도커허브ID>/kubia**

```
docker tag kubia simto49/kubia 
```

아래 이미지를 보면 kubia 라는 이미지와 simto49/kubia라는 이미지의 ID가 동일한 것을 확인할 수 있습니다.\
동일한 이미지의 태그가 생성된 것을 알 수 있습니다.\
![스크린샷 2019-04-28 오후 6.54.48.png](https://nhnent.dooray.com/share/posts/7abiS3cCRWOmBEZOXX32sw/files/2467436135182402792)

로그인을 한 이후 이미지를 push 해봅니다.

```
$docker login simt49
$docker push simto49/kubia 
```

올라간 이미지를 찾아볼까요?

```
$docker search simto49
```

태그된 이미지를 확인할 수 있습니다.

위 이미지는 Kubernetes in action의 저자가 이미 dockerHub에 올려놨으므로 앞으로는 저자가 올린 이미지를 사용하겠습니다.

### 쿠버네티스 첫번째 앱 실행

```
$kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1
```

로그에 보면 API 버전에 대해서 deprecate 된다고 설명하고 있는데 위의 pod 생성하는 API는 replicationController를 사용해서 Pod를 생성합니다.\
\--generator=run-pod/v1 으로 생성하면 replicationController를 통해서 생성되지 않기 때문에 Pod 관리 영역을 설명하는데 어려움이 있으므로 우선 상기 옵션으로 진행합니다.

Pod를 관리하는 방법은 여러가지(deployment, replicaset , replicationController 등 ) 가 있는데 이 부분은 나중에 설명하도록 하겠습니다.

#### 생성된 Pod 확인

```
$kubectl get pods
```

**실제 Pod내의 컨테이너에서 동작하는지 봐야겠죠?**

```
$kubectl exec -it kubia-kubia-dbfj2 /bin/bash
root@kubia-dbfj2# curl localhost:8080
you've hit kubia-dbfj2
```

**아직 외부에 서비스를 노출하기 위한 방법 ( expose ) 에 대해서 논의하지 않았기 때문에 특정 pod의 동작 여부를 테스트하고 싶을 때 사용하는 방법을 통해 서비스 동작 여부를 확인해보겠습니다.**

```
$kubectl port-forward kubia-dbfj2 8888:8080
```

![스크린샷 2019-04-28 오후 8.05.39.png](https://nhnent.dooray.com/share/posts/7abiS3cCRWOmBEZOXX32sw/files/2467471720907712211)\
포트 포워드 프로세스를 띄웠으므로 별도의 창에서 확인하면 정상 동작하는 것을 확인할 수 있다.

```
$curl localhost:8888 
 You've hit kubia-dbfj2
```

#### 화면 뒤에 숨겨진 프로세스

* 도커 이미지를 제작해 도커 허브에 푸시 한다.
* kubectl 명령을 통해 쿠버네티스 API 서버에 REST HTTP 요청을 전송해 클러스터 새로운 ReplicationController 객체를 만든다.
* ReplicationController는 새로운 포드를 생성하고 스케줄러에 의해 워커 노드들 중 하나테 스케줄 된다.
* 해당 노드는 kubelet에 의해 포드가 스케줄 된 것을 확인하고 도커가 이미지가 로컬에 없음을 확인하고 허브에서 지정된 이미지를 가지고 오도록 명령했다.
* 이미지를 다운로드 한 후에 도커가 컨테이너를 생성하고 실행한다.

**우선 하나 삭제해봅니다.**

```
$kubectl delete pod kubia-dbfj2
```

**삭제가 됐다는데 다른 컨테이너가 떠있네요?**

**힌트 : ReplicationController**

**ReplicationController를 통해 스케일 아웃해보겠습니다.**

```
$kubectl scale --replicas=3 rc/kubia 
```

**replication Controller 한테 원하는 상태( 복제 수를 3으로 세팅) 를 알려준다.**

**생성된 Pod들의 정보를 좀 더 알고 싶을 때 -o (출력 옵션) wide**

```
$kubectl get pods -o wide 
```

**각각의 Pod들은 그들이 할당 받은 IP를 통해서 통신할 수 있는 걸 확인할 수 있습니다.**

**이제 이 Pod들을 서비스로 노출을 한번 해볼까요?**

**Pod를 외부에 노출하는 방식으로는 ClusterIP, LoadBalancer, NodePort, ExternalName 등이 있는데 세부 사항은 서비스에 대해서 설명할 때 다시 하겠습니다.**

**우선은 ClusterIP 타입으로 클러스터 내부에만 오픈되는 타입으로 노출시키겠습니다.**

**서비스로 Pod를 묶으면 loadbalancing 기능도 사용할 수 있습니다.**

```
$ kubectl expose rc kubia --type=ClusterIP --name kubia-http
service/kubia-http exposedku
$kubectl get svc 
```

**서비스의 Cluster IP 가 할당된 부분을 확인할 수 있습니다.**

**ClusterIP는 클러스터내부에서만 동작하는 IP로 외부 IP와 연동을 통해 외부 환경에 노출됩니다.**

**minikube VM 환경에서, Pod 내의 컨테이너에서 접근 되는지 확인해볼까요?**

### Kubernetes Building Blocks

**kubernetes Object 모델에 대해서 알아보고 그런 오브젝트들을 그룹핑하는 역할을 담당하는 Labels 와 Selector에 대해서 알아보겠습니다.**

#### Kubernetes Object Model

쿠버네티스는 매우 풍부한 객체 모델을 가지고 있는데 그 모델들은 쿠버네티스에서 아래와 같은 영구적인 실체를 표현합니다.

* 실행중인 컨테이너형 어플리케이션이 무엇인지, 어떤 노드에서 실행 중인지
* 애플리케이션 사용량
* 재시작/ 업그레이드 정책, Fault Tolerance 등과 같은 다양한 정책 들

각각의 오브젝트들은 Spec 필드를 통해서 원하는 형상(예를 들어 Pod의 경우 컨테이너의 목록, 볼륨 ) 에 대해서 정의합니다.\
쿠버네티스 시스템은 그 오브젝트의 state 필드를 관리하고 그 형상을 기록합니다. 쿠버테티스 컨트롤 플레인은 해당 오브젝트의 실제 상태와 원하는 상태에 대해서 감시하고 두 상태를 동일하게 만들기 위해 노력합니다.

쿠버네티스 객체를 생성하기 위해서는 쿠버네티스 API 서버에 사양 필드 ( Spec ) 에 기본정보와 해당 오브젝트가 원하는 상태 에 대해서 기술되어있어야 하고 JSON 형태로 있어야 합니다.\
대개는 Yaml 파일로 기술하며 형상은 비슷합니다.

예를 들어서 Deployment Object에의 예를 한번 보면

```
apiVersion: apps/v1                     # 접근하려는 API 서버의  API endpoint 
kind: Deployment                        # 어떤 Object or Resouce 인지. 
metadata:                                    # 네임, 라벨, 주석 외 그밖의 것들 
  name: nginx-deployment
  labels:
    app: nginx
spec:                                            # 원하는 배포 상태 정의
  replicas: 3                                  # 복제할 Pod는 3개  spec.template.spec에서 제공하는 템플릿 스펙으로 Pod를 만듬
  selector:
    matchLabels:
      app: nginx
  template:                                   # Pod 복제시 사용될 Pod의 Spec 
    metadata:
      labels:
        app: nginx
    spec:                                           
      containers:                            # Pod 내에 동작할 컨테이너들 정보 
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

\


$ kubectl describe Object명 -o yaml 형태로 확인이 가능합니다. \
$ kubectl edit Obejct 명 -o yaml 명령어로 수정할 수도 있습니다.&#x20;

#### 앞서 만들어놓은 오브젝트들에 대해서 확인해볼까요? 나중에 설명들을 때 유용합니다.

```
$ kubectl describe pod kubia-j7cct 
$ kubectl describe svc kubia-http
$ kubectl describe rc kubia
$ kubectl edit rc kubia
```

#### Object 모델에는 Pod, ReplicaSet, Deployments, NameSpace 등이 있습니다. 하나씩 알아가보죠.

### Pods

**가장 작은, 간단한 쿠버네티스 오브젝트입니다.**

**배포의 단위가 되며 어플리케이션의 하나의 인스턴스를 표현합니다.**

**하나의 Pod는 하나 이상의 컨테이너들의 논리적 집합으로 동일한 호스트에서 동작해야 하며 , 동일한 네트워크 네임스페이스/ 스토리지을 공유합니다**

**언제든 죽을 수 있는 단명( Node fault , 컨테이너 행 등 )하는 객체이기 때문에 스스로 컨트롤( 재 시작 등)을 할 수 있는 능력이 없습니다. 우리가 복제, self-healing 등을 위해서 Deployments, ReplicaSet, RePlicationController 등같은 Controller들을 사용하는 이유가 되겠죠.**

### ReplicationController

**rc 라고도 부르는 컨트롤러로 마스터 노드의 컨트롤러 매니저의 한 부분입니다.**

**어느 때라도 pod 의 지정된 복제 수가 실행되고 있도록 하는 역할을 담당합니다.**

**일반적으로 문제가 발생했을 때 자체적으로 재시작하는 능력이 없는 Pod를 독립적으로 배포하지 않기 때문에 Pod를 생성하거나 관리하기 위해서 Controller Object들을 사용합니다.**

### ReplicaSets

**ReplicationController의 다음 세대로 Pod를 선택하는 label Select 방식으로 equality ( = , != ) , set-based selectors( in, notin, exist ) 를 제공합니다**

**Repilcation Controller는 equality 만 제공합니다. 그외는 동일함**

#### 하나의 Pod가 죽었을 때 동작 방식

### Deployments

**Pods와 ReplicaSet들의 선언적 업데이트를 담당합니다.**

**마스터 노드의 컨트롤러 매니저의 부분인 DeploymentController 에 의해서 원하는 상태와 일치됩니다.**

#### 롤 아웃 방식 배포

**Deployment 내의 Pods Template의 컨테이너 버전만을 업데이트함으로서 배포**

### Namesapaces

**수많은 조직/팀/프로젝트 들과 같이 사용자들을 나누고 싶을 때 클러스터들을 파티셔닝하는 방법으로 Namespace를 사용해서 서브 클러스터를 제공할 수 있습니다.**

**$kubectl get namespaces**

**kube-system ( 쿠버네티스 자체 오브젝트들을 위한 네임스페이스 )**

**default ( 아무나\~ 접근 , 기본 )**

**Namespace 를 기준으로 리소스를 나눌 수 있기 때문에 리소스 제약에 사용합니다.**

### Labels

**어떤 쿠버네티스 오브젝트들에도 할당할 수 있는 Key-Value pair입니다.**

**오브젝트들의 서브집합을 구성하거나 선택하는데 사용할 수 있습니다.**

### Label Selectors

**equality ( = , != ) , set-based selectors( in, notin, exist )**

### Network Setup Challenges

#### 다른 실습을 하기 전에 한번은 얘기해야할 네트워크에 대해서 훍어보기만 해보겠습니다.

**쿠버네티스 클러스터의 기능을 최대한으로 활용하기 위해 다음과 같은 부분이 해결되어야 합니다.**

**각 Pod는 하나의 유니크한 IP를 가지고 있어야 한다.**

**Pod 안의 컨테이너들은 서로 커뮤니케이션이 가능해야 한다.**

**Pod는 클러스터 내 다른 Pod들과 커뮤니케이션이 가능해야 한다.**

**Pod 안에 배포된 어플리케이션은 외부에서 접근이 가능해야 한다.**

**어떻게 해결했는지를 확인해보며 쿠버네티스 내부 구성에 대해서 좀 더 알게 될 겁니다. 자세한 세부사항은 좀 더 나중에 확인해보겠습니다.**

### Assigning a Unique Address to Each Pod

#### Kubernetes 각 Pod에 IP 주소를 할당하기 위해 CNI( Container Network Interface)를 사용합니다.

**컨테이너 런타임은 IP 할당을 Bridge / MACvlan 같은 Plugin과 연결된 CNI에 일임합니다.**

**CRI가 요청하는 시점에 Plugin을 통해 IP를 할당하고 CRI한테 IP를 전달합니다.**

### Container-to-Container Communication Inside a Pod

#### OS 시스템의 도움으로 모든 컨테이너 런타임은 각각의 컨테이너를 위한 독립적인 네트워크를 생성하는데 Linux에서는 network namespaces라고 합니다.

#### 이 네트워크는 Host OS 혹은 컨테이너들간에 공유됩니다.

#### 그래서 각각의 컨테이너들은 localhost를 통해 도달할 수 있습니다.

### Pod-to-Pod Communication Across Nodes

#### 클러스터 환경에서 Pod는 어떤 노드에서도 실행되도록 스케줄 될 수 있기 때문에 해당 Pod는 Nod간의 커뮤니케이션도 가능해야 합니다.

#### 그 말은 Host들 간에 어떤 NAT를 통하지 말아야 한다는 얘기이며 이를 해결하기 위해서 아래와 같은 방법을 사용합니다.

* **GKE같이 물리적인 인프라를 사용하여 Routing이 가능한 Pods와 Nodes를 사용**
* **Software Define Network( like Flannel, Weave, Calico 등 )을 사용함**

####

