# M1 사용자를 위한 Docker Lima



### lima란 <a href="#lima-eb-9e-80" id="lima-eb-9e-80"></a>

lima는 자동파일공유와 포트 포워딩(WSL2과 유사)을 지원하는 linux vm과 컨테이너를 제공한다.

> lima는 일종의 비공식적인 "맥을 위한 컨테이너"라 보면 된다.

✅ 자동 파일 공유\
✅ 자동 포트 포워딩\
✅ 컨테이너 기능 탑재

[lima 공식 깃허브](https://github.com/lima-vm/lima)



{% file src="../.gitbook/assets/M1 사용자를 위한 Docker Lima.zip" %}



### lima 설치 방법 <a href="#lima-ec-84-a4-ec-b9-98-eb-b0-a9-eb-b2-95" id="lima-ec-84-a4-ec-b9-98-eb-b0-a9-eb-b2-95"></a>

#### homebrew를 이용한 설치 <a href="#homebrew-eb-a5-bc-ec-9d-b4-ec-9a-a9-ed-95-9c-ec-84-a4-ec-b9-98" id="homebrew-eb-a5-bc-ec-9d-b4-ec-9a-a9-ed-95-9c-ec-84-a4-ec-b9-98"></a>

```
brew install lima
```

docker와 docker-compose가 깔려있지 않다면 같이 설치한다

```
brew install lima docker docker-compose
```

homebrew 없이 설치를 하려 한다면... [수동 설치 매뉴얼](https://github.com/lima-vm/lima#install-lima)

#### lima가 제대로 설치 됐는지 확인 <a href="#lima-ea-b0-80-ec-a0-9c-eb-8c-80-eb-a1-9c-ec-84-a4-ec-b9-98-eb-90-90-eb-8a-94-ec-a7-80-ed-99-95-ec-9d" id="lima-ea-b0-80-ec-a0-9c-eb-8c-80-eb-a1-9c-ec-84-a4-ec-b9-98-eb-90-90-eb-8a-94-ec-a7-80-ed-99-95-ec-9d"></a>

```
limactl
```



\


### lima에 docker 설치 방법 및 명령어 종류 <a href="#lima-ec-97-90-docker-ec-84-a4-ec-b9-98-eb-b0-a9-eb-b2-95-eb-b0-8f-eb-aa-85-eb-a0-b9-ec-96-b4-ec-a2-8" id="lima-ec-97-90-docker-ec-84-a4-ec-b9-98-eb-b0-a9-eb-b2-95-eb-b0-8f-eb-aa-85-eb-a0-b9-ec-96-b4-ec-a2-8"></a>

#### docker + lima config 파일 <a href="#docker-2b-lima-config-ed-8c-8c-ec-9d-bc" id="docker-2b-lima-config-ed-8c-8c-ec-9d-bc"></a>

해당 프로필 스펙은 다음과 같다

* OS: Ubuntu 21.10 (Impish Indri)
* CPU: 4 cores
* Memory: 4 GiB
* Disk: 100 GiB
* Mounts: \~ (read-only), /tmp/lima (writable)
* SSH: 127.0.0.1:60022

**docker가 추가된 docker.yaml 환경설정 파일**

```
# Example to use Docker instead of containerd & nerdctl
# $ limactl start ./docker.yaml
# $ limactl shell docker docker run -it -v $HOME:$HOME --rm alpine

# To run `docker` on the host (assumes docker-cli is installed):
# $ export DOCKER_HOST=$(limactl list docker --format 'unix://{{.Dir}}/sock/docker.sock')
# $ docker ...

# This example requires Lima v0.8.0 or later
images:
# Hint: run `limactl prune` to invalidate the "current" cache
- location: "https://cloud-images.ubuntu.com/impish/current/impish-server-cloudimg-amd64.img"
  arch: "x86_64"
- location: "https://cloud-images.ubuntu.com/impish/current/impish-server-cloudimg-arm64.img"
  arch: "aarch64"
mounts:
- location: "~"
- location: "/tmp/lima"
  writable: true
# containerd is managed by Docker, not by Lima, so the values are set to false here.
containerd:
  system: false
  user: false
provision:
- mode: system
  # This script defines the host.docker.internal hostname when hostResolver is disabled.
  # It is also needed for lima 0.8.2 and earlier, which does not support hostResolver.hosts.
  # Names defined in /etc/hosts inside the VM are not resolved inside containers when
  # using the hostResolver; use hostResolver.hosts instead (requires lima 0.8.3 or later).
  script: |
    #!/bin/sh
    sed -i 's/host.lima.internal.*/host.lima.internal host.docker.internal/' /etc/hosts
- mode: system
  script: |
    #!/bin/bash
    set -eux -o pipefail
    command -v docker >/dev/null 2>&1 && exit 0
    export DEBIAN_FRONTEND=noninteractive
    curl -fsSL https://get.docker.com | sh
    # NOTE: you may remove the lines below, if you prefer to use rootful docker, not rootless
    systemctl disable --now docker
    apt-get install -y uidmap dbus-user-session
- mode: user
  script: |
    #!/bin/bash
    set -eux -o pipefail
    systemctl --user start dbus
    dockerd-rootless-setuptool.sh install
    docker context use rootless
probes:
- script: |
    #!/bin/bash
    set -eux -o pipefail
    if ! timeout 30s bash -c "until command -v docker >/dev/null 2>&1; do sleep 3; done"; then
      echo >&2 "docker is not installed yet"
      exit 1
    fi
    if ! timeout 30s bash -c "until pgrep rootlesskit; do sleep 3; done"; then
      echo >&2 "rootlesskit (used by rootless docker) is not running"
      exit 1
    fi
  hint: See "/var/log/cloud-init-output.log". in the guest
hostResolver:
  # hostResolver.hosts requires lima 0.8.3 or later. Names defined here will also
  # resolve inside containers, and not just inside the VM itself.
  hosts:
    host.docker.internal: host.lima.internal
portForwards:
- guestSocket: "/run/user/{{.UID}}/docker.sock"
  hostSocket: "{{.Dir}}/sock/docker.sock"
message: |
  To run `docker` on the host (assumes docker-cli is installed), run the following commands:
  ------
  docker context create lima --docker "host=unix://{{.Dir}}/sock/docker.sock"
  docker context use lima
  docker run hello-world
  ------
```

다른 예시 파일들은 여기서 확인할수 있다 : [lima/examples](https://github.com/lima-vm/lima/tree/master/examples)

#### docker.yaml을 이용한 설치 <a href="#docker.yaml-ec-9d-84-ec-9d-b4-ec-9a-a9-ed-95-9c-ec-84-a4-ec-b9-98" id="docker.yaml-ec-9d-84-ec-9d-b4-ec-9a-a9-ed-95-9c-ec-84-a4-ec-b9-98"></a>

docker를 이용하기 위해 docker.yaml 파일을 이용하여 lima를 실행시킨다.\
해당 yaml 파일이 있는 곳으로 이동한 다음에 파일을 실행시킨다.

_(첨부파일의 docker.yaml 파일을 다운받으면 된다.)_

```
limactl start docker.yaml
```

설치가 완료되기까지 5분 정도의 시간이 걸린다.\


#### 실행중인 리스트 확인하기 <a href="#ec-8b-a4-ed-96-89-ec-a4-91-ec-9d-b8-eb-a6-ac-ec-8a-a4-ed-8a-b8-ed-99-95-ec-9d-b8-ed-95-98-ea-b8-b0" id="ec-8b-a4-ed-96-89-ec-a4-91-ec-9d-b8-eb-a6-ac-ec-8a-a4-ed-8a-b8-ed-99-95-ec-9d-b8-ed-95-98-ea-b8-b0"></a>

```
limactl list
```

설치한 docker vm 이 잘 떠있는 것을 볼 수 있다.

```
NAME      STATUS     SSH                ARCH       CPUS    MEMORY    DISK      DIR
docker    Running    127.0.0.1:51363    aarch64    4       4GiB      100GiB    /Users/taejun/.lima/docker
```

\
\


### lima 접속 및 docker 실행 명령어 <a href="#lima-ec-a0-91-ec-86-8d-eb-b0-8f-docker-ec-8b-a4-ed-96-89-eb-aa-85-eb-a0-b9-ec-96-b4" id="lima-ec-a0-91-ec-86-8d-eb-b0-8f-docker-ec-8b-a4-ed-96-89-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

다음 명령어를 입력하여 lima vm으로 접속한다

```
limactl shell docker
```

(_이후에 나오는 명령어들은 해당 lima vm 에 접속하여 진행하는 단계들이다_)

접속후 아래의 명령어를 입력하여 docker 가 잘 실행되는지 확인한다.

```
docker
```



#### 간단한 docker 명령어 <a href="#ea-b0-84-eb-8b-a8-ed-95-9c-docker-eb-aa-85-eb-a0-b9-ec-96-b4" id="ea-b0-84-eb-8b-a8-ed-95-9c-docker-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

| command                                        | description                                                                  | Reference                                                                                                                                  |
| ---------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `docker-compose up [OPTIONS]`                  | <p>서비스를 위한 컨테이너를 빌드, (재)생성, 시작 및 연결<br>docker-compose.yml 파일의 위치에서 실행 가능</p> | [https://docs.docker.com/compose/reference/up/](https://docs.docker.com/compose/reference/up/)                                             |
| `docker ps [OPTIONS]`                          | 컨테이너 조회                                                                      | [https://docs.docker.com/engine/reference/commandline/ps/](https://docs.docker.com/engine/reference/commandline/ps/)                       |
| `docker images [OPTIONS] [REPOSITORY[:TAG]]`   | 이미지 조회                                                                       | [https://docs.docker.com/engine/reference/commandline/images/](https://docs.docker.com/engine/reference/commandline/images/)               |
| `docker volume ls [OPTIONS]`                   | 볼륨 조회                                                                       | [https://docs.docker.com/engine/reference/commandline/volume\_ls/](https://docs.docker.com/engine/reference/commandline/volume\_ls/)       |
| `docker-compose down`                          | 컨테이너 중지, `up`에서 생성한 컨테이너, 네트워크, 볼륨 및 이미지 제거                                  | [https://docs.docker.com/compose/reference/down/](https://docs.docker.com/compose/reference/down/)                                         |
| `docker rm [OPTIONS] CONTAINER [CONTAINER...]` | 하나 이상의 컨테이너 제거                                                               | [https://docs.docker.com/engine/reference/commandline/rm/](https://docs.docker.com/engine/reference/commandline/rm/)                       |
| `docker rmi [OPTIONS] IMAGE [IMAGE...]`        | 호스트 노드에서 하나 이상의 이미지를 제거(및 태그 해제)                                             | [https://docs.docker.com/engine/reference/commandline/rmi/](https://docs.docker.com/engine/reference/commandline/rmi/)                     |
| `docker volume prune [OPTIONS]`                | 사용하지 않는 모든 로컬 볼륨 제거                                                          | [https://docs.docker.com/engine/reference/commandline/volume\_prune/](https://docs.docker.com/engine/reference/commandline/volume\_prune/) |
| `docker exec -it [컨테이너명 or 컨테이너ID] bash`      | 실행 중인 컨테이너에 접속                                                               | [https://docs.docker.com/engine/reference/commandline/exec/](https://docs.docker.com/engine/reference/commandline/exec/)                   |

\
\


### lima에서 docker-compose 사용하기 <a href="#lima-ec-97-90-ec-84-9c-docker-compose-ec-82-ac-ec-9a-a9-ed-95-98-ea-b8-b0" id="lima-ec-97-90-ec-84-9c-docker-compose-ec-82-ac-ec-9a-a9-ed-95-98-ea-b8-b0"></a>

#### docker-compose.yml 란? <a href="#docker-compose.yml-eb-9e-80-3f" id="docker-compose.yml-eb-9e-80-3f"></a>

> 연결된 다수의 컨테이너를 하나로 통합하여 관리하는 도구

docker-compose 에서는 컨테이너 실행에 사용되는 옵션과 컨테이너 간 의존성을 모두 docker-compose.yml 파일에 적어두고, docker-compose 명령어를 사용하여 컨테이너를 실행 및 관리한다.

docker-compose에 대한 자세한 내용이 궁금하다면 : [Docker Compose와 버전별 특징](https://meetup.toast.com/posts/277)\
\


#### lima에서 docker-compose 설치하기 <a href="#lima-ec-97-90-ec-84-9c-docker-compose-ec-84-a4-ec-b9-98-ed-95-98-ea-b8-b0" id="lima-ec-97-90-ec-84-9c-docker-compose-ec-84-a4-ec-b9-98-ed-95-98-ea-b8-b0"></a>

docker-compose를 사용하기 위해서는 lima vm환경에 설치하여야 한다. 다음의 명령어를 입력한다.

```
sudo apt install docker-compose
```

설치 후 아래의 명령어를 입력하여 docker-compose가 제대로 설치되었는지 확인한다.

```
docker-compose
```

제대로 설치되었으면 아래와 같은 화면이 뜬다.\
![docker-compose 설치.png](https://nhnent.dooray.com/wikis/2341059147520508271/files/3226742523075670938)\
\


#### docker-compose.yml 항목 <a href="#docker-compose.yml-ed-95-ad-eb-aa-a9" id="docker-compose.yml-ed-95-ad-eb-aa-a9"></a>

docker-compose파일은 다음과 같은 항목들로 구성되어있다.

| 항목                | example                                                                                                              | description                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `version:`        | `version: '3'`                                                                                                       | <p>3만 쓰게 되면 가장 최신 버전으로 인식.<br>구체적으로 버전을 명시할 수 있다.</p> |
| `services:`       |                                                                                                                      | 서비스들에 대한 정보 명시                                        |
| `container_name:` | `container_name: "local-mongodb"`                                                                                    | 컨테이너명 명시                                              |
| `image:`          | `image: mongo`                                                                                                       | 가져올 이미지 명시                                            |
| `restart:`        | `restart: always`                                                                                                    | 컨테이너 종료 -> 재실행                                        |
| `environment:`    | `environment: MONGO_INITDB_DATABASE: local-db MONGO_INITDB_ROOT_USERNAME: root MONGO_INITDB_ROOT_PASSWORD: password` | 컨테이너의 환경변수 명시                                         |
| `ports:`          | `ports: - "27017:27017"`                                                                                             | 포트 매핑 정보 명시. (호스트:컨테이너)                               |
| `volumes:`        | `volumes: - "local-mongodb:/var/lib/mongo"`                                                                          | 볼륨 매핑 (호스트:컨테이너)                                      |

\


### docker-comopse로 mongodb, mysql, redis 환경 세팅하기 <a href="#docker-comopse-eb-a1-9c-mongodb-2c-mysql-2c-redis-ed-99-98-ea-b2-bd-ec-84-b8-ed-8c-85-ed-95-98-ea-b8" id="docker-comopse-eb-a1-9c-mongodb-2c-mysql-2c-redis-ed-99-98-ea-b2-bd-ec-84-b8-ed-8c-85-ed-95-98-ea-b8"></a>

본 예시에서는 mongodb, mysql, redis 를 포함하는 컨테이너를 docker-comopse를 사용하여 설치한다.

#### docker-compose up으로 컨테이너 띄우기 <a href="#docker-compose-up-ec-9c-bc-eb-a1-9c-ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-eb-9d-84-ec-9a-b0-ea-b8-b0" id="docker-compose-up-ec-9c-bc-eb-a1-9c-ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-eb-9d-84-ec-9a-b0-ea-b8-b0"></a>

docker-compose 파일이 있는 위치로 이동하여 해당 파일을 실행시킨다.\
_(첨부파일의 docker-compose.yml 파일을 다운받으면 된다.)_

```
docker-compose up -d
```

(_-d 옵션: 백그라운드에서 돌아갈수 있도록 한다._)

해당 명령어는 파일이름이 docker-compose.yml인 경우에 사용가능하고, 파일 이름이 다른 경우라면 다음의 명령어를 사용하면 된다.

```
docker-compose -f docker-compose-test.yml up
docker-compose -f {파일이름} up
```

(_-f 옵션: 기본으로 제공하는 docker-compose.yml이 아닌 별도의 파일명을 실행할 때 사용_)

제대로 컨테이너가 띄워졌다면 다음과 같은 화면을 볼 수 있다.\
![docker-compose-up.png](https://nhnent.dooray.com/wikis/2341059147520508271/files/3226742523098000123)

#### docker image 확인하기 <a href="#docker-image-ed-99-95-ec-9d-b8-ed-95-98-ea-b8-b0" id="docker-image-ed-99-95-ec-9d-b8-ed-95-98-ea-b8-b0"></a>

다음의 명령어를 입력하여 docker image를 확인할수 있다.

```
docker images -a
```

(_-a 옵션: 실행중이지 않더라도 전체 리스트를 보여줌_)

방금전 docker-compose up을 하면서 다운받은 mongo, redis, mysql 이미지가 있는것을 확인할 수 있다.\


#### docker process 확인하기 <a href="#docker-08process-ed-99-95-ec-9d-b8-ed-95-98-ea-b8-b0" id="docker-08process-ed-99-95-ec-9d-b8-ed-95-98-ea-b8-b0"></a>

다음의 명령어를 입력하여 docker에 떠있는 process를 확인할수 있다.

```
docker ps -a
```

(_-a 옵션: 실행중이지 않더라도 전체 리스트를 보여줌_)

방금전 docker-compose up으로 실행시킨 mongo, redis, mysql 이미지가 있는것을 확인할 수 있다.\


#### docker-compose down으로 컨테이너 내리기 <a href="#docker-compose-down-ec-9c-bc-eb-a1-9c-ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-eb-82-b4-eb-a6-ac-ea-b8-b0" id="docker-compose-down-ec-9c-bc-eb-a1-9c-ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-eb-82-b4-eb-a6-ac-ea-b8-b0"></a>

다음의 명령어를 입력하면 해당 파일로 띄운 컨테이너를 내릴 수 있다.

```
docker-compose down
```

### mongodb의 docker-compose.yml <a href="#mongodb-ec-9d-98-docker-compose.yml" id="mongodb-ec-9d-98-docker-compose.yml"></a>

```
version: '3'

services:
  local-mongodb:
    container_name: "local-mongodb"
    image: mongo
    restart: always 
    environment:
      MONGO_INITDB_DATABASE: local-db
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: password
    ports:
      - "27017:27017"
    volumes:
      - "local-mongodb:/var/lib/mongo"

volumes:
  local-mongodb:
```

#### 간단한 MongoDB 명령어 <a href="#ea-b0-84-eb-8b-a8-ed-95-9c-mongodb-eb-aa-85-eb-a0-b9-ec-96-b4" id="ea-b0-84-eb-8b-a8-ed-95-9c-mongodb-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

| command                           | example                                                                                                                            | description      |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `mongo admin -u [유저명] -p [비밀번호]`  | `mongo admin -u root -p password`                                                                                                 | mongodb 접속       |
| `use [데이터베이스명]`                   | `use mydb`                                                                                                                         | 데이터베이스 생성 & 사용   |
| `show dbs`                        |                                                                                                                                    | 데이터베이스 조회        |
| `db`                              |                                                                                                                                    | 현재 사용중인 DB 조회    |
| `db.createCollection("[컬렉션명]")`   | `db.createCollection("person")`                                                                                                    | collection 생성    |
| `show collections`                |                                                                                                                                    | collection 조회    |
| `db.[컬렉션명].insert([JSON format])` | `db.person.insert({"name" : "hyoung", "email" : "[joonhyuck-hyoung@nhn-commerce.com](mailto:joonhyuck-hyoung@nhn-commerce.com)"})` | document(데이터) 생성 |
| `db.[컬렉션명].find()`                | `db.person.find()`                                                                                                                 | document 조회      |
| `db.[컬렉션명].remove([JSON format])` | `db.person.remove({"name":"hyoung"})`                                                                                              | document 삭제      |
| `db.[컬렉션명].drop()`                | `db.person.drop()`                                                                                                                 | collection 삭제    |

\
\


### mysql의 docker-compose.yml <a href="#mysql-ec-9d-98-docker-compose.yml" id="mysql-ec-9d-98-docker-compose.yml"></a>

```
local-mysql:
    image: mysql/mysql-server:8.0.23
    container_name: local-mysql
    hostname: local-mysql
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: accounts
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_DATABASE: authorization
    volumes:
      - "local-mysql:/var/lib/mysql"
```

\
\


#### lima에서 docker로 띄운 mysql 접속하기 <a href="#lima-ec-97-90-ec-84-9c-docker-eb-a1-9c-eb-9d-84-ec-9a-b4-mysql-ec-a0-91-ec-86-8d-ed-95-98-ea-b8-b0" id="lima-ec-97-90-ec-84-9c-docker-eb-a1-9c-eb-9d-84-ec-9a-b4-mysql-ec-a0-91-ec-86-8d-ed-95-98-ea-b8-b0"></a>

***

docker mysql container 접속: `$ docker exec -it local-mysql bash`

mysql 접속: `$ mysql -h localhost -P 13306 -u root -p authorization`

### redis의 docker-compose.yml <a href="#redis-ec-9d-98-docker-compose.yml" id="redis-ec-9d-98-docker-compose.yml"></a>

```
local-redis:
    container_name: local-redis
    hostname: local-redis
    image: redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - "local-redis:/var/lib/"
```

#### lima에서 docker로 띄운 redis 접속하기 <a href="#lima-ec-97-90-ec-84-9c-docker-eb-a1-9c-eb-9d-84-ec-9a-b4-redis-ec-a0-91-ec-86-8d-ed-95-98-ea-b8-b0" id="lima-ec-97-90-ec-84-9c-docker-eb-a1-9c-eb-9d-84-ec-9a-b4-redis-ec-a0-91-ec-86-8d-ed-95-98-ea-b8-b0"></a>

***

docker redis container 접속: `$ docker exec -it local-redis redis-cli`\


\
\


### docker volume - mount <a href="#docker-volume---mount" id="docker-volume---mount"></a>

* 도커 컨테이너에서 작성되거나 수정된 파일은 컨테이너가 파기되면 호스트도 함께 삭제된다.
* 호스트쪽 파일 시스템에 데이터를 마운트하면 컨테이너에서 삭제해도 호스트 파일 시스템에 남아있게 된다.
* volume 종류

1. Bind Mount\
   호스트 환경의 특정 경로를 컨테이너 내부 볼륨 경로와 연결하여 마운트한다. 이 방법은 보안에 영향을 미칠 수 있음
2. Volume (가장 일반적)\
   도커 볼륨은 도커 컨테이너에서 도커 내부에 도커 엔진이 관리하는 볼륨을 생성\
   생성된 볼륨은 호스트 디렉터리의 /var/lib/docker/volumes 경로에 저장되며, 도커를 사용하여 관리가 용이
3. tmpfs Mount\
   호스트의 파일 시스템이 아닌, 메모리에 저장하는 방식을 사용

\


### Volume mount 테스트 <a href="#volume-mount-ed-85-8c-ec-8a-a4-ed-8a-b8" id="volume-mount-ed-85-8c-ec-8a-a4-ed-8a-b8"></a>

* **사전준비**\
  mysql image 가 존재하는 docker-compose.yml 파일

```
version: '3'

services:
  local-mysql:
    image: mysql/mysql-server:8.0.23
    container_name: local-mysql
    hostname: local-mysql
    restart: always
    ports:
      - "13306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_DATABASE: mydb
    volumes:
      - "local-mysql:/var/lib/mysql"

volumes:
  local-mysql:
```

\


#### 1.MySQL <a href="#1.mysql" id="1.mysql"></a>

1. mysql image 가 존재하는 docker-compose.yml 을 통해 컨테이너 빌드\
   `docker-compose up -d --f`
2. 실행중인 mysql 컨테이너에 접속\
   `docker exec -it [컨테이너명 or 컨테이너ID] bash`
3. mysql 접근\
   `mysql -u [유저명] -p [비밀번호]`
4. 데이터베이스 사용\
   `use [데이터베이스명];`
5. 테스트를 위한 테이블 (member\_table) 생성

```
CREATE TABLE member_table (
 seq        INT NOT NULL AUTO_INCREMENT,
 mb_id     VARCHAR(20),
 mb_pw    VARCHAR(20),
 address   VARCHAR(50),
 mb_tell    VARCHAR(50),
  PRIMARY KEY(seq)
)
```

\
6\. 더미데이터 추가\
`INSERT INTO member_table(mb_id, mb_pw, address, mb_tell) VALUES("test", "test", "test", "test");`\
7\. 데이터 확인\
`SELECT * FROM member_table;`

```
+-----+-------+-------+---------+---------+
| seq | mb_id | mb_pw | address | mb_tell |
+-----+-------+-------+---------+---------+
|   1 | test  | test  | test    | test    |
|   2 | test  | test  | test    | test    |
|   3 | test  | test  | test    | test    |
|   4 | test  | test  | test    | test    |
|   5 | test  | test  | test    | test    |
|   6 | test  | test  | test    | test    |
|   7 | test  | test  | test    | test    |
|   8 | test  | test  | test    | test    |
|   9 | test  | test  | test    | test    |
|  10 | test  | test  | test    | test    |
|  11 | test  | test  | test    | test    |
|  12 | test  | test  | test    | test    |
+-----+-------+-------+---------+---------+
12 rows in set (0.00 sec)
```

\
8\. mysql 접근 종료\
`exit`

#### 2. volume mount 확인 <a href="#2.-08volume-mount-ed-99-95-ec-9d-b8" id="2.-08volume-mount-ed-99-95-ec-9d-b8"></a>

1. docker volume 생성 확인\
   `docker volume ls`

```
DRIVER    VOLUME NAME
local     docker_local-mysql
```

\
2\. lima 환경 내의 물리적 저장장소 확인\
`cd ~/.local/share/docker/volumes/`\
`cd _data`\
접근이 거부되면 `sudo -s`\
3\. 테스트용으로 생성한 테이블 (member\_table) 확인

root@lima-default:/home/taejun.linux/.local/share/docker/volumes/docker\_local-mysql/\_data/mydb# `ls`\
**member\_table.ibd**

\


#### mount 과정 확인 방법 <a href="#mount-ea-b3-bc-ec-a0-95-ed-99-95-ec-9d-b8-eb-b0-a9-eb-b2-95" id="mount-ea-b3-bc-ec-a0-95-ed-99-95-ec-9d-b8-eb-b0-a9-eb-b2-95"></a>

1. `docker-compose down` => `docker-compose up` 이후에도 lima 위에 데이터가 존재하는지 여부 확인
2. `docker rm -f [컨테이너명]` 을 통해 컨테이너를 지워도 lima 위에 데이터가 존재하는지 여부 확인

### 부록 <a href="#eb-b6-80-eb-a1-9d" id="eb-b6-80-eb-a1-9d"></a>

#### 의문점 <a href="#ec-9d-98-eb-ac-b8-ec-a0-90" id="ec-9d-98-eb-ac-b8-ec-a0-90"></a>

mongodb의 데이터는 mysql와 다르게 지정된 volume이 아닌 해시값으로 이루어진 폴더에 저장된다.

```
taejun@lima-default:~/.local/share/docker/volumes$ cd cea1548f8644bcb41ac708a0b3d204f2616f20a35159f25b42aed9645b6d6197/_data/
taejun@lima-default:~/.local/share/docker/volumes/cea1548f8644bcb41ac708a0b3d204f2616f20a35159f25b42aed9645b6d6197/_data$ ls
WiredTiger         collection-0-4752117110476890269.wt  index-3-4752117110476890269.wt  mongod.lock
WiredTiger.lock    collection-2-4752117110476890269.wt  index-5-4752117110476890269.wt  sizeStorer.wt
WiredTiger.turtle  collection-4-4752117110476890269.wt  index-6-4752117110476890269.wt  storage.bson
WiredTiger.wt      collection-7-4752117110476890269.wt  index-8-4752117110476890269.wt
WiredTigerHS.wt    diagnostic.data                      index-9-4752117110476890269.wt
_mdb_catalog.wt    index-1-4752117110476890269.wt       journal
```

\


#### 기존 프로젝트의 docker-compose 파일을 사용할 경우 겪을 수 있는 어려움 <a href="#ea-b8-b0-ec-a1-b4-ed-94-84-eb-a1-9c-ec-a0-9d-ed-8a-b8-ec-9d-98-docker-compose-ed-8c-8c-ec-9d-bc-ec-9" id="ea-b8-b0-ec-a1-b4-ed-94-84-eb-a1-9c-ec-a0-9d-ed-8a-b8-ec-9d-98-docker-compose-ed-8c-8c-ec-9d-bc-ec-9"></a>

기존의 docker image들은 arm64/v8 기반으로 이미지가 생성되어있다. 그렇기에 기존 이미지를 이용하여 컨테이너를 생성하게 될 경우 m1 플랫폼 환경인 linux/x86\_64 를 지원하지 않기 때문에 docker-comopse up을 할 경우 에러가 발생한다.

다음과 같은 방법으로 해당 에러를 해결할수 있었다.

1. docker-compose file 에 다음의 내용을 추가한다.

```
platform: linux/x86_64
```

이 방법은 해당 이미지가 이미 buildx옵션으로 빌드되어 도커 허브에 존재하는 경우에만 통한다.

2\. 해당 이미지를 **docker buildx**를 이용하여 m1 플랫폼도 지원할수 있도록 재빌드 하여 사용하는 방법\
linux/x86\_64 도 지원하는 멀티플랫폼 빌드와 관련된 내용은 다음의 링크에서 확인할 수 있다.

[Docker buildx build(at M1 Macbook)](https://velog.io/@inyong\_pang/Devops-Docker-buildx-at-M1-Macbook)

\
이제 해결이 안되는 경우에는 구글링으로...

#### 추가 진행할 내용 <a href="#ec-b6-94-ea-b0-80-ec-a7-84-ed-96-89-ed-95-a0-eb-82-b4-ec-9a-a9" id="ec-b6-94-ea-b0-80-ec-a7-84-ed-96-89-ed-95-a0-eb-82-b4-ec-9a-a9"></a>

현재 redis, mongo, mysql은 m1환경에서 lima를 이용하여 docker-compose 파일을 이용하여 컨테이너를 띄웠지만, kafka + zookeepr의 경우에는 docker-compose 파일을 이용한 컨테이너를 띄우는 것에 어려움을 겪고있다.

해당 부분은 추가적으로 성공하는 데로 본 내용에 추가할 예정이다.

docker-compose를 이용하여 컨테이너를 띄우게 되면 다음과 같은 에러가 나면서 프로세스가 떠있지 못하고 자꾸 죽는 현상을 겪고 있다.

```
standard_init_linux.go:228: exec user process caused: exec format error
```

출처 : Cho Yoohwa, Kang Sunghyuk, Hyoung Joonhyuck
