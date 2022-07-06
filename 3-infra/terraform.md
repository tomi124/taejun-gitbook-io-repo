#  terraform



## 1. Terraform 이란? <a href="#1-terraform" id="1-terraform"></a>

* 여러 클라우드에 리소스를 배포할 수 있도록 API를 호환해 놓은 오픈 소스 tool
* 서비스 실행에 필요한 환경을 구축하는 도구라는 측면에 있어서 Chef, Ansible과 같은 프로비저닝 도구로써 분류됨
* 선언형 언어 HCL (Hashicorp Configuration Language)로 인프라 구성을 작성
* 테라폼이 선언형 언어를 분석 및 클라우드 프로바이더의 API를 호출

## 2. Terraform 특성 <a href="#2-terraform" id="2-terraform"></a>

### (1) Infrastructure as Code <a href="#1-infrastructure-as-code" id="1-infrastructure-as-code"></a>

* 인프라를 코드로 정의하여 생산성과 투명성을 높일 수 있음
* 정의한 코드를 쉽게 공유할 수 있어 효율적으로 협업 가능

### (2) Execution Plan <a href="#2-execution-plan" id="2-execution-plan"></a>

* 변경 계획과 변경 적용을 분리하여 변경 내용을 적용할 때 발생할 수 있는 실수를 줄일 수 있음

### (3) Resource Graph <a href="#3-resource-graph" id="3-resource-graph"></a>

* 사소한 변경이 인프라 전체에 어떤 영향을 미칠지 미리 확인 가능
* 종속성 그래프를 작성하여 이 그래프를 바탕으로 계획을 세우고, 이 계획을 적용했을 때 변경되는 인프라 상태를 확인할 수 있음

### (4) Change Automation <a href="#4-change-automation" id="4-change-automation"></a>

* 여러 장소에 같은 구성의 인프라를 구축하고 변경할 수 있도록 자동화할 수 있음
* 인프라를 구축하는 데 드는 시간을 절약 가능

## 3. Terraform Registry <a href="#3-terraform-registry" id="3-terraform-registry"></a>

* 클라우드 플랫폼 제어를 위해 필요한 플러그인을 다운받기 위한 [저장소](https://registry.terraform.io/)
* 500 개 이상의 서비스 프로바이더 지원
* 서비스 프로바이더의 신뢰성에 따라 3개의 Tier로 구분 (Official, Verified, Community)

![](https://velog.velcdn.com/images/chl4651/post/843a52a3-c918-44de-ab13-6132cad3d823/image.png)

## 4. HCL (Hashicorp Configuration Language) <a href="#4-hcl-hashicorp-configuration-language" id="4-hcl-hashicorp-configuration-language"></a>

* Hashicorp 사에서 개발한 선언형 언어 (Json-Format)
* Terraform에서 리소스를 선언하는 용도로 사용

```json
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}

# Block: HCL 에서 나타내는 객체의 컨테이너 (e.g. Resource, Variable 등)
# Argument: 블록 내에서 변수에 값을 할당함
# Expression: 문자 그대로 혹은 다른 값을 참조하여 값을 나타냄
```

## 5. Terraform 구조 <a href="#5-terraform" id="5-terraform"></a>

### (1) 구성 모듈 <a href="#1" id="1"></a>

#### (a) Terraform CLI <a href="#a-terraform-cli" id="a-terraform-cli"></a>

* 테라폼 명령을 처리하기 위한 기본 명령줄 도구
* Terraform Core 기능 포함: HCL 분석, 그래프 생성

#### (b) HCL (Hashicorp Configuration Language) <a href="#b-hcl-hashicorp-configuration-language" id="b-hcl-hashicorp-configuration-language"></a>

* .tf 확장자를 가진 선언형 언어 파일
* 프로비저닝할 리소스를 선언

#### (c) Terraform State File <a href="#c-terraform-state-file" id="c-terraform-state-file"></a>

* .tfstate 확장자를 가진 상태 파일
* terraform 동작이 수행되었을 때 자동으로 생성되며 테라폼 명령의 실행 결과에 따른 상태정보 관리를 위해 사용됨

#### (d) Terraform Plugin (Provider) <a href="#d-terraform-plugin-provider" id="d-terraform-plugin-provider"></a>

* 사용하는 Cloud Platform (e.g. AWS, Azure)에 따라 동적으로 설치
* "terraform init" 명령 실행 시에 동작으로 다운로드됨

![](https://velog.velcdn.com/images/chl4651/post/0aacd3df-d74f-4124-8a7d-df2919c8c65a/image.png)

### (2) Core vs. Plugins <a href="#2-core-vs-plugins" id="2-core-vs-plugins"></a>

* Terraform 특성 상, 모든 프로바이더를 설치 시점부터 지원하기에는 비효율적임
* Terraform Plugin (Binary)를 동적으로 설치하는 방식으로 Architecture 구성\
  ![](https://velog.velcdn.com/images/chl4651/post/dbc92557-8fde-4d7a-a5a2-fac1a68a764c/image.png)

### (3) Terraform 동작 흐름 <a href="#3-terraform" id="3-terraform"></a>

* 테라폼은 크게 4단계의 동작흐름으로 구성되어 있음

#### (a) terraform init <a href="#a-terraform-init" id="a-terraform-init"></a>

* Working Directory에 필요한 Terraform 환경 구성
* HCL 파일 내 리소스 내용을 기준으로 필요한 Terraform Plugin 모듈 설치

#### (b) terraform plan <a href="#b-terraform-plan" id="b-terraform-plan"></a>

* 실제 프로비저닝 작업 이전에 플랜 생성
* HCL 코드가 어떤 인프라를 만들게 될지 예측 결과를 미리 보여주는 단계

#### (c) terraform apply <a href="#c-terraform-apply" id="c-terraform-apply"></a>

* HCL 파일 내용을 실제 인프라 환경에 적용 (Infrastructure as a Code)

#### (d) terraform destroy <a href="#d-terraform-destroy" id="d-terraform-destroy"></a>

* Terraform Apply 이후에 배포한 리소스를 삭제하고 싶은 경우에 사용
