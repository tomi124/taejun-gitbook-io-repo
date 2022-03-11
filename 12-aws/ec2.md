# EC2

## AWS EC2 <a href="#aws-ec2" id="aws-ec2"></a>

AWS EC2란 Amazon Elastic Compute Cloud(Amazon EC2)로써 C가 2개여서 EC2로 불린다.\
말그대로 Amazon에서 제공하는 탄력있는 클라우트 컴퓨터이다.\
EC2에서는 다음과 같은 기능을 제공한다

* 인스턴스: 가상 컴퓨팅 환경
* Amazon 머신 이미지(AMI): 서버에 필요한 운영체제와 여러 소프트웨어들이 적절히 구성된 상태로 제공되는 템플릿으로 인스턴스를 쉽게 만들 수 있습니다.
* 인스턴스 유형: 인스턴스를 위한 CPU, 메모리, 스토리지, 네트워킹 용량의 여러 가지 구성 제공
* 키 페어를 사용하여 인스턴스 로그인 정보 보호(AWS는 퍼블릭 키를 저장하고 사용자는 개인 키를 안전한 장소에 보관하는 방식)
* 인스턴스 스토어 볼륨: 임시 데이터를 저장하는 스토리지 볼륨으로 인스턴스 종료 시 삭제됨
* Amazon Elastic Block Store(Amazon EBS), 즉 Amazon EBS 볼륨을 사용해 영구 스토리지 볼륨에 데이터 저장
* 인스턴스와 Amazon EBS 볼륨 등의 리소스를 다른 물리적 장소에서 액세스할 수 있는 리전 및 가용 영역
* 보안 그룹을 사용해 인스턴스에 연결할 수 있는 프로토콜, 포트, 소스 IP 범위를 지정하는 방화벽 기능
* 탄력적 IP 주소(EIP): 동적 클라우드 컴퓨팅을 위한 고정 IPv4 주소
* 태그: 사용자가 생성하여 Amazon EC2 리소스에 할당할 수 있는 메타데이터
* AWS 클라우드에서는 논리적으로 격리되어 있지만, 원할 때 마다 고객의 네트워크와 간편히 연결할 수 있는 가상 네트워크, Virtual Private Clouds(VPC)

## AWS Management Console <a href="#aws-management-console" id="aws-management-console"></a>

AWS가입 및 신용카드 등록을 사전에 완료하면, AWS의 Management Console에 접속할 수 있다.

![](https://media.vlpt.us/post-images/inyong\_pang/0a7ff710-2fc6-11ea-8298-5bb2e4a1ff0f/image.png)

## EC2 Instance <a href="#ec2-instance" id="ec2-instance"></a>

#### 1. EC2 인스턴스를 선택한다 <a href="#1-ec2" id="1-ec2"></a>

![](https://media.vlpt.us/post-images/inyong\_pang/31fc9b90-2fc6-11ea-86b3-238eec66dc4f/image.png)

#### 2. "Launch Instance" 를 선택한다. <a href="#2-launch-instance" id="2-launch-instance"></a>

![](https://media.vlpt.us/post-images/inyong\_pang/3a4e46e0-2fc6-11ea-8298-5bb2e4a1ff0f/image.png)

#### 3. Ubunt 서버를 선택한다. <a href="#3-ubunt" id="3-ubunt"></a>

![](https://media.vlpt.us/post-images/inyong\_pang/40837ad0-2fc6-11ea-8298-5bb2e4a1ff0f/image.png)

#### 4. Instance type을 선택한다. t2\_micro가 1년 동안 무료임으로 t2\_micro를 선택한 후 "Next: Configure Instance Details"를 선택한다 <a href="#4-instance-type-t2_micro-1-t2_micro-next-configure-instance-details" id="4-instance-type-t2_micro-1-t2_micro-next-configure-instance-details"></a>

![](https://media.vlpt.us/post-images/inyong\_pang/49a854a0-2fc6-11ea-86b3-238eec66dc4f/image.png)

#### 5. "Auto-assign Public IP" 설정을 "Enable"로 한다. <a href="#5-auto-assign-public-ip-enable" id="5-auto-assign-public-ip-enable"></a>

나머지는 default 값을 그대로 나두면 된다. 이제 "Review and Luanch"를 클릭하자.

![](https://media.vlpt.us/post-images/inyong\_pang/53aa9620-2fc6-11ea-8f58-c75abca5fa3b/image.png)

#### 6. 설정이 제대로 되었는지 확인 한 후 "Launch" 버튼을 누른다. <a href="#6-launch" id="6-launch"></a>

![](https://media.vlpt.us/post-images/inyong\_pang/5b7fe580-2fc6-11ea-86b3-238eec66dc4f/image.png)

#### 7. Pem 키 파일을 설정한다. "Create a new key pair" 를 선택한후 download 받는다. 그리고 "Launch Instance"를 클릭한다 <a href="#7-pem-create-a-new-key-pair-download-launch-instance" id="7-pem-create-a-new-key-pair-download-launch-instance"></a>

pem키는 한번 발급후 다시 받을 수 없음으로 download 후 관리를 잘 해야한다.

![](https://media.vlpt.us/post-images/inyong\_pang/7ce7d660-2fc6-11ea-8f58-c75abca5fa3b/image.png)

#### 8. 이제 EC2 인스턴스가 생성 될것이다. 준비 되는데 몇 분 이상 소요 될 수 있다. 인스턴스가 생성이 되면 Public DNS 와 Public IP 주소를 확인한다. <a href="#8-ec2-public-dns-public-ip" id="8-ec2-public-dns-public-ip"></a>

![](https://media.vlpt.us/post-images/inyong\_pang/82298290-2fc6-11ea-8f58-c75abca5fa3b/image.png)

#### 9. Public IP 주소로 ssh 접속을 한다. <a href="#9-public-ip-ssh" id="9-public-ip-ssh"></a>

```bash
ssh -i path/to/pem ubuntu@13.125.228.11
```

path/to/pem 은 앞서 7번에서 다운로드 받은 pem 키 파일의 경로이다.
