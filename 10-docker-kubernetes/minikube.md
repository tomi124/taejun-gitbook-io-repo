# Minikube

## minikube

### 설치

```
brew install minikube
```

### 에러

```
$ minikube
zsh: command not found: minikube
```

```
brew unlink minikube
brew link minikube
```

### 버전 확인

```
$ minikube version
minikube version: v1.23.2
commit: 0a0ad764652082...
```

### 커맨드

| 커맨드                  | 개요                         |
| -------------------- | -------------------------- |
| minikube start       | 가상 머신 기동, kubectl 사용이 가능해짐 |
| minikube stop        | 가상 머신 정지                   |
| minikube delete      | 가상 머신 삭제                   |
| minikube status      | 상태 출력                      |
| minikube ip          | 가상 머신의 IP 주소 표시            |
| minikube ssh         | 가상 머신의 리눅스에 로그인            |
| minikube addons list | 애드온 기능 목록                  |
| minikube dashboard   | 브라우저를 기동하여 쿠버네티스의 대시보드 접속  |

### 대시보드 기동

```
$ minikube addons enable metrics-server

$ minikube dashboard
```

metrics-server 를 기동하고 그래프가 표시될 때까지는 약 5분 정도의 시간이 걸린다

### 메모리 8GB로 기동

```
minikube start --memory=8192
```

### 미니쿠베 기동 확인

```
kubectl get po --all-namespaces
```

### EFK 개시

```
minikube addons enable efk
```

### NodePort 번호 확인

```
kubectl get svc --all-namespaces
```

### 미니쿠베 서비스 목록

```
minikube service list
```

### Links

* [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

### Docs

* [vagrant](https://jace.link/open/vagrant)
* [kubectl](https://jace.link/open/kubectl)

***

### 관련 문서

* [15단계로 배우는 도커와 쿠버네티스](https://jace.link/open/15%EB%8B%A8%EA%B3%84%EB%A1%9C\_%EB%B0%B0%EC%9A%B0%EB%8A%94\_%EB%8F%84%EC%BB%A4%EC%99%80\_%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4)

\
