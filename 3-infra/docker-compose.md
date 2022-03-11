# Docker-compose



## Docker Compose <a href="#docker-compose" id="docker-compose"></a>

* Docker Compose란?
* 주요 명령어
* Docker Compose 구성해보기
* Docker Compose Environment
* Compose Network 설정 옵션
* docker Compose Version별 yml 변수
* 버전별 특징 및 구성

### Docker Compose란? <a href="#docker-compose-eb-9e-80-3f" id="docker-compose-eb-9e-80-3f"></a>

Docker Compose는 다중 컨테이너 애플리케이션을 정의하고 공유할 수 있도록 개발된 도구로

YAML 파일을 통하여 단일 명령을 사용하여 모두 실행하거나 모두 종료가 가능하며

YAML 파일에 애플리케이션 스택을 정의하고 프로젝트 리포지토리 루트에 파일을 저장함으로

다른 사용자가 Repository를 복제하고 `Compose` 실행만으로 구동이 가능하다.

\


#### 단일 호스트의 여러 격리 된 환경 <a href="#eb-8b-a8-ec-9d-bc-ed-98-b8-ec-8a-a4-ed-8a-b8-ec-9d-98-ec-97-ac-eb-9f-ac-ea-b2-a9-eb-a6-ac-eb-90-9c-e" id="eb-8b-a8-ec-9d-bc-ed-98-b8-ec-8a-a4-ed-8a-b8-ec-9d-98-ec-97-ac-eb-9f-ac-ea-b2-a9-eb-a6-ac-eb-90-9c-e"></a>

Compose는 프로젝트 이름을 사용하여 환경을 서로 격리하고 여러 다른 컨텍스트에서 이 프로젝트 이름을 사용하여 접근한다.

실행시 `-p` 옵션을 통하여 프로젝트 이름 변경이 가능하며 Default 값은 프로젝트 폴더 이름이 된다.

`docker-dompose up -p {프로젝트 이름}`

#### &#x20; <a href="#ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-ec-83-9d-ec-84-b1-ec-8b-9c-eb-b3-bc-eb-a5-a8-eb-8d-b0-ec-9d-b4-e" id="ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-ec-83-9d-ec-84-b1-ec-8b-9c-eb-b3-bc-eb-a5-a8-eb-8d-b0-ec-9d-b4-e"></a>

#### 컨테이너 생성시 볼륨 데이터 보존 <a href="#ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-ec-83-9d-ec-84-b1-ec-8b-9c-eb-b3-bc-eb-a5-a8-eb-8d-b0-ec-9d-b4-e" id="ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-ec-83-9d-ec-84-b1-ec-8b-9c-eb-b3-bc-eb-a5-a8-eb-8d-b0-ec-9d-b4-e"></a>

컨테이너 생성시 볼륨 데이터 보존하여 데이터가 휘발되지 않도록 처리한다.

\


#### 변경된 컨테이너 만 재생성 <a href="#eb-b3-80-ea-b2-bd-eb-90-9c-ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-eb-a7-8c-ec-9e-ac-ec-83-9d-ec-84-b1" id="eb-b3-80-ea-b2-bd-eb-90-9c-ec-bb-a8-ed-85-8c-ec-9d-b4-eb-84-88-eb-a7-8c-ec-9e-ac-ec-83-9d-ec-84-b1"></a>

컨테이너를 만드는 데 사용되는 구성을 캐시하여 변경되지 않은 서비스를 다시 시작하면 Compose는 기존 컨테이너를 다시 사용한다.

\


#### 변수 및 환경 간 구성 이동 <a href="#eb-b3-80-ec-88-98-eb-b0-8f-ed-99-98-ea-b2-bd-ea-b0-84-ea-b5-ac-ec-84-b1-ec-9d-b4-eb-8f-99" id="eb-b3-80-ec-88-98-eb-b0-8f-ed-99-98-ea-b2-bd-ea-b0-84-ea-b5-ac-ec-84-b1-ec-9d-b4-eb-8f-99"></a>

Compose 파일의 변수를 지원

변수를 사용하여 다양한 환경 또는 다른 사용자에 맞게 컴포지션 커스텀이 가능하다.

### 주요 명령어 <a href="#ec-a3-bc-ec-9a-94-eb-aa-85-eb-a0-b9-ec-96-b4" id="ec-a3-bc-ec-9a-94-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

#### up -d <a href="#up--d" id="up--d"></a>

`docker-compose.yml` 파일의 내용에 따라 이미지를 빌드하고 서비스를 실행한다.

**up으로 compose를 실행시 단계별 진행사항**

1. 서비스를 띄울 네트워크 설정
2. 필요한 볼륨 생성(혹은 이미 존재하는 볼륨과 연결)
3. 필요한 이미지 풀(pull)
4. 필요한 이미지 빌드(build)
5. 서비스 의존성에 따라 서비스 실행

**options**

* `-d`: 서비스 백그라운드로 실행. (docker run에서의 -d와 같음)
* `--force-recreate`: 컨테이너를 지우고 새로 생성.
* `--build`: 서비스 시작 전 이미지를 새로 생성

#### ps <a href="#ps" id="ps"></a>

현재 환경에서 실행 중인 각 서비스의 상태를 표시한다.

```
   Name                  Command               State           Ports
---------------------------------------------------------------------------
tj_app_1     docker-entrypoint.sh sh -c ...   Exit 1
tj_mysql_1   docker-entrypoint.sh mysqld      Up       3306/tcp, 33060/tcp
```

#### stop, start <a href="#stop-2c-start" id="stop-2c-start"></a>

서비스를 멈추거나, 멈춰 있는 서비스를 시작한다.

```
$ docker-compose stop
Stopping tj_mysql_1 ... done

$ docker-compose start
Starting app   ... done
Starting mysql ... done
```

#### down <a href="#down" id="down"></a>

서비스를 삭제한다.\
컨테이너와 네트워크를 삭제하며, 옵션에 따라 볼륨도 삭제한다.

```
$ docker-compose down
Stopping tj_mysql_1 ... done
Removing tj_app_1   ... done
Removing tj_mysql_1 ... done
Removing network tj_default
```

**options**

* `--volume`: 볼륨까지 같이 삭제 (DB 데이터 초기화 하는데 용이함)

#### exec <a href="#exec" id="exec"></a>

실행 중인 컨테이너에서 명령어를 실행한다.

```
$ docker-compose exec django ./manage.py makemigrations

$ docker-compose exec db psql postgres postgres
```

#### run <a href="#run" id="run"></a>

docker run 과 마찬 가지로 특정 명령어를 일회성으로 실행한다.\
`exec`는 프로세서를 실행시켜놓을때 사용되고 `run`은 batch성 작업에 사용 특화 된것으로 보여진다.

**options**

* `--rm` : 해당 명령어가 종료된 뒤 컨테이너도 종료

#### logs <a href="#logs" id="logs"></a>

output으로 나온 log들을 확인 할때 사용한다.\
docker의 logs와 마찬가지로 `--follow`, `-f`를 하여 실시간으로 나오는 로그 확인이 가능하다.

```
$ docker-compose logs -f
Attaching to docker_app_1, docker_mysql_1
app_1    | yarn install v1.22.5
app_1    | info No lockfile found.
app_1    | [1/4] Resolving packages...
app_1    | [2/4] Fetching packages...
app_1    | [3/4] Linking dependencies...
app_1    | [4/4] Building fresh packages...
app_1    | success Saved lockfile.
app_1    | Done in 0.14s.
app_1    | yarn run v1.22.5
app_1    | error Couldn't find a package.json file in "/app"
app_1    | info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
mysql_1  | 2021-01-05 04:43:03+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.32-1debian10 started.
```

**options**

* `-f`, `--follow`: 실시간 로그 출력

#### config <a href="#config" id="config"></a>

`docker-compose`에 최종적으로 적용된 설정을 보여준다.\
`-f`를 이용하여 여러개의 설정파일을 띄웠을때 확인에 용이함

```
$ docker-compose config
services:
  app:
    command: sh -c "yarn install && yarn run dev"
    environment:
      MYSQL_DB: todos
      MYSQL_HOST: mysql
      MYSQL_PASSWORD: secret
      MYSQL_USER: root
    image: node:12-alpine
    ports:
    - published: 3000
      target: 3000
    volumes:
    - /Users/tj/project/tj/tj-msa/docker:/app:rw
    working_dir: /app
  mysql:
    environment:
      MYSQL_DATABASE: todos
      MYSQL_ROOT_PASSWORD: secret
    image: mysql:5.7
    volumes:
    - todo-mysql-data:/var/lib/mysql:rw
version: '3.8'
volumes:
  todo-mysql-data: {}
```

### Docker Compose 구성해보기. <a href="#docker-compose-ec-84-a4-ec-a0-95-eb-b0-a9-eb-b2-95" id="docker-compose-ec-84-a4-ec-a0-95-eb-b0-a9-eb-b2-95"></a>

docker compose 파일은 크게 아래와 같이 구성되어있다.

```
version: "3.8" # docker-compose 설정파일 버전
services:
  web:
    # 웹 애플리케이션 설정
  db:
    # 데이터베이스 설정
networks:
  # 네트워크 설정
volumes:
  # 볼륨 설정
```

> \
>
>
> docker-compose version 확인 링크
>
> [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

### 서비스 정의 <a href="#ec-84-9c-eb-b9-84-ec-8a-a4-ec-a0-95-ec-9d-98" id="ec-84-9c-eb-b9-84-ec-8a-a4-ec-a0-95-ec-9d-98"></a>

#### App Service 정의 <a href="#app-service-ec-a0-95-ec-9d-98" id="app-service-ec-a0-95-ec-9d-98"></a>

```
docker run -dp 3000:3000 \
  -w /app -v ${PWD}:/app \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

상단의 `Docker`실행명령어를 아래처럼 `Docker Compose`로 작성이 가능하다.

```
version: "3.8"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos 
```

Compose  장점으로는 상대경로 입력이 가능하며 yaml로 구성하여 설정파일의 가독성이 좋아진다.

#### MySQL 서비스 정의 <a href="#mysql-ec-84-9c-eb-b9-84-ec-8a-a4-ec-a0-95-ec-9d-98" id="mysql-ec-84-9c-eb-b9-84-ec-8a-a4-ec-a0-95-ec-9d-98"></a>

```
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

```
version: "3.8"

services:
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

#### 최종본 <a href="#ec-b5-9c-ec-a2-85-eb-b3-b8" id="ec-b5-9c-ec-a2-85-eb-b3-b8"></a>

```
version: "3.8"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

`docker-compose.yml` 파일명으로 YAML 파일을 작성하고 `docker-compose up` 명령어를 통하여 시작한다.

\
백그라운드 실행 옵션 `-d`

```
docker-compose up -d
```

> 만약 `docker-compose.yml`이 아닌 다른 파일을 실행하기 위해선\
> `docker-compose -f {YAML_FILE_PATH} up` 형식으로 명령이 가능하다.
>
> `ex) $ docker-compose -f docker-compose-custom.yml up`\
> `docker-compose -f docker-compose.yml -f docker-compose-test.yml up` 형태로 두개도 가능하다.\
> YAML을 두개 이상 설정할 경우 앞에 있는 설정보다 뒤에 있는 파일이 우선시 된다.

### Docker Compose Environment <a href="#docker-compose-environment" id="docker-compose-environment"></a>

#### `.env` file <a href="#file" id="file"></a>

환경변수 설정으로 아래와 같이 참조가 가능하다.

```
web:
  image: "webapp:${TAG}"
```

기본적으로 `docker-compose up`을 한 상태에서는 `.env` 파일을 찾아 파일 내부에 있는 값을 환경 변수로 사용한다.

```
$ cat .env
TAG=v1.5

$ cat docker-compose.yml
version: '3'
services:
  web:
    image: "webapp:${TAG}"$

$ docker-compose config
version: '3'
services:
  web:
    image: 'webapp:v1.5'


$ export TAG=v2.0
$ docker-compose config

version: '3'
services:
  web:
    image: 'webapp:v2.0'
```

여러 파일에서 동일한 환경 변수를 설정할 때 사용할 값을 선택하기 위해 Compose에서 사용하는 우선 순위는 다음과 같다.

1. Compose file
2. Shell environment variables
3. Environment file
4. Dockerfile
5. Variable is not defined

#### 상황에 따른 `.env` 적용법 <a href="#ec-83-81-ed-99-a9-ec-97-90-eb-94-b0-eb-a5-b8-ec-a0-81-ec-9a-a9-eb-b2-95" id="ec-83-81-ed-99-a9-ec-97-90-eb-94-b0-eb-a5-b8-ec-a0-81-ec-9a-a9-eb-b2-95"></a>

`dev`, `prod`, `local` 환경에 따라서\
`.env.dev`, `.env.prod`, `.env.local` 형식으로 할당을 줄 수 있는데\
`--env-file` 옵션으로 environment 파일 지정이 가능하다.

```
$ docker-compose --env-file ./config/.env.dev up
```

#### CLI Environment <a href="#cli-environment" id="cli-environment"></a>

아래의 명령어들은 각각 그 아래의 compose 파일이 일치하게 맵핑된다.

```
$ docker run -e VARIABLE1

web:
  environment:
    - VARIABLE1
```

```
$ docker-compose run -e DEBUG=1 web python console.py

web:
  environment:
    - DEBUG=1
```

#### Docker Cli Environment Variable <a href="#docker-cli-environment-variable" id="docker-cli-environment-variable"></a>

CLI에서 실행할때의 참고 변수 키 값\
[https://docs.docker.com/compose/reference/envvars/](https://docs.docker.com/compose/reference/envvars/)

### Compose Network 설정 옵션 <a href="#compose-network-ec-84-a4-ec-a0-95-ec-98-b5-ec-85-98" id="compose-network-ec-84-a4-ec-a0-95-ec-98-b5-ec-85-98"></a>

```
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```

위 Compose를 실행했을때의 네트워크가 생성되는 과정은 아래와 같다.

1. `myapp_default` 네트워크 생성
2. `web` 컨테이너가 생성. (`web`이라는 이름으로 `myapp_default`에 접속)
3. `db` 컨테이너가 생성. (`db`이라는 이름으로 `myapp_default`에 접속)

* 네트워크가 구성된 이후 `postgres://db:5432` 주소로 접속 가능
* 내부 컨테이너 끼리는 `postgres://db:5432`로 접속하고 외부에선 `postgres://{DOCKER_IP}:8001`로 접속
* 컨테이너 업데이트를 위해 다시 시작할 경우에는 컨테이너 이름은 같지만 다른 IP로 네트워크가 생성되어\
  IP 베이스는 IP를 다시 찾아서 연결하여야 하고 다른 컨테이너에선 기존에 접속했던 컨테이너 이름으로 재 연결을 시도하여야 한다.

#### Links <a href="#links" id="links"></a>

다른 서비스의 컨테이너에 연결한다.\
이름만 지정하거나 `{name}:{alias}`형식으로 지정할 수 있다.\
`depend_on` 처럼 디펜던시 관계가 맺어지며 네트워크랑 같이 쓰일때는 하나 이상의 네트워크가 관리되어 네트워크만 사용하길 권장

```
version: "3"
services:
  web:
    build: .
    links:
      - "redis"
      - "db:database"
  db:
    image: postgres
```

#### Custom Network <a href="#custom-network" id="custom-network"></a>

Custom Network를 통해 좀 더 복잡한 네트워크 토폴로지를 만들 수 있다

```
version: "3"
services:
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend
networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```

`top-level network` 옵션에서 커스텀하게 지정이 가능

위와 같이 설정을 하면 `app`에서만 `proxy`, `db`에 접근 가능하고 `db`와 `proxy`는 서로 접근이 불가능하게 설정이된다.

#### IPv4, IPv6 <a href="#ipv4-2c-ipv6" id="ipv4-2c-ipv6"></a>

IPv6 사용하려면 `enable_ipv6`를 반드시 `true`로 해야한다.

```
version: "2.4"

services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
        - subnet: 172.16.238.0/24
          gateway: 172.16.238.1
        - subnet: 2001:3984:3989::/64
          gateway: 2001:3984:3989::1
```

#### Default Network <a href="#default-network" id="default-network"></a>

default를 이용하여 자체 네트워크를 지정하면서 앱 전체의 기본적인 네트워크 구성이 가능하다.

```
version: "3"
services:

  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres

networks:
  default:
    # Use a custom driver
    driver: custom-driver-1
```

#### Use Pre Existing Network <a href="#use-pre-existing-network" id="use-pre-existing-network"></a>

기존의 외부의 네트워크를 사용하려면 다음과 같이 `external`을 사용하여 명시한다

```
networks:
  default:
    external:
      name: my-pre-existing-network
```

### docker Compose Version 별 yml 변수 <a href="#docker-compose-version-eb-b3-84-yml-eb-b3-80-ec-88-98" id="docker-compose-version-eb-b3-84-yml-eb-b3-80-ec-88-98"></a>

#### Versioning <a href="#versioning" id="versioning"></a>

`docker-compose.yml` 상단에 작성하는 versioning 작성 방법

* `Version 1`에서는 버저닝을 생략
* `Version 2`부터 마이너 버전(2.x)까지 설정해야 함 (생략시 2.0으로 적용된다.)
* `Version 3`은 도커 스웜과 같이 사용되도록 디자인 됨

#### 각 docker 버전 별 사용 가능한 `docker-compose.yml` 버전 <a href="#ea-b0-81-docker-eb-b2-84-ec-a0-84-eb-b3-84-ec-82-ac-ec-9a-a9-ea-b0-80-eb-8a-a5-ed-95-9c-eb-b2-84-ec" id="ea-b0-81-docker-eb-b2-84-ec-a0-84-eb-b3-84-ec-82-ac-ec-9a-a9-ea-b0-80-eb-8a-a5-ed-95-9c-eb-b2-84-ec"></a>

| Compose file format | Docker Engine release | Compose Version |
| :-----------------: | :-------------------: | :-------------: |
|         3.8         |        19.03.0+       |     1.26.0+     |
|         3.7         |        18.06.0+       |        -        |
|         3.6         |        18.02.0+       |        -        |
|         3.5         |        17.12.0+       |        -        |
|         3.4         |        17.09.0+       |        -        |
|         3.3         |        17.06.0+       |        -        |
|         3.2         |        17.04.0+       |        -        |
|         3.1         |        1.13.1+        |        -        |
|         3.0         |        1.13.0+        |     1.10.0+     |
|         2.4         |        17.12.0+       |     1.21.0+     |
|         2.3         |        17.06.0+       |     1.16.0+     |
|         2.2         |        1.13.0+        |     1.13.0+     |
|         2.1         |        1.12.0+        |      1.9.0+     |
|         2.0         |        1.10.0+        |      1.6.0+     |
|         1.0         |        1.9.1.+        |                 |

#### Compose 버전에 따른 차이점 <a href="#compose-eb-b2-84-ec-a0-84-ec-97-90-eb-94-b0-eb-a5-b8-ec-b0-a8-ec-9d-b4-ec-a0-90" id="compose-eb-b2-84-ec-a0-84-ec-97-90-eb-94-b0-eb-a5-b8-ec-b0-a8-ec-9d-b4-ec-a0-90"></a>

* Structure가 허용하는 키 값과 매개변수
* 실행해야하는 최소 Docker Engine 버전
* 네트워킹과 관련된 Compose의 동작

#### Compose 마이그레이션 <a href="#compose-eb-a7-88-ec-9d-b4-ea-b7-b8-eb-a0-88-ec-9d-b4-ec-85-98" id="compose-eb-a7-88-ec-9d-b4-ea-b7-b8-eb-a0-88-ec-9d-b4-ec-85-98"></a>

**Version 2.x -> 3.x 마이그레이션**

Compose 버전 `2`와 `3`의 구조는 동일하지만 삭제된 매개변수들이 있어서 `3.X`에 맞게 수정하여준다.

* `volume_driver`

```
version: "3.8"
services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data
volumes:
  data:
    driver: mydriver
```

* 서비스에서 볼륨 드라이버를 설정하는 대신 `top-level volumes option`을 사용하여 볼륨을 정의 하고 드라이버를 지정
* `volumes_from`: 서비스간에 볼륨을 공유하려면 `top-level volumes option`을 사용하여 정의하고 서비스 수준 볼륨 옵션을 사용하여 공유하는 각 서비스에서 참조
* `cpu_shares`, `cpu_quota`, `cpuset`, `mem_limit`, `memswap_limit`는 `3.x`의 `resource`로 대체
  * 해당 옵션은 docker swarm에만 적용
    * 컨테이너에 적용하기 위해선 아래 링크와 같이 적용 필요
      * [https://docs.docker.com/compose/compose-file/compose-file-v2/#cpu-and-other-resources](https://docs.docker.com/compose/compose-file/compose-file-v2/#cpu-and-other-resources)
      * [https://github.com/docker/compose/issues/4513](https://github.com/docker/compose/issues/4513)

```
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

* `extends`은 제거되어 다른 방법을 찾아봐야함
  * [https://docs.docker.com/compose/extends/#extending-services](https://docs.docker.com/compose/extends/#extending-services)
* `group_add`은 `3.x`에서 제거됨
* `pids_limit`은 `3.x`에 도입되지 않음
* networks - `link_local_ips` 옵션은 `3.x`에 도입되지 않음

**Version 1 -> 2.x 마이그레이션**

* 모든것을 한단계씩 들여쓰기를 하고 `services:`를 삽입
* 최상단에 version: `2.x` 지정
* `dockerfile`은 `build` 하위로 옮긴다.

```
build:
  context: .
  dockerfile: Dockerfile-alternate
```

* `log_driver`, `log_opt`는 `logging` 하위로 두고 다음과 같이 변경한다.

```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

* `net`는 `network_mode`로 대체

```
net: host    ->  network_mode: host
net: bridge  ->  network_mode: bridge
net: none    ->  network_mode: none

net: "container:web"  ->  network_mode: "service:web"

# net: "container:[container name/id]" 형태는 값을 변경할 필요가 없음
net: "container:cont-name"  ->  network_mode: "container:cont-name"
net: "container:abc12345"   ->  network_mode: "container:abc12345"
```

* `volumes` named volumes이 있는 경우 `top-level volumes option`에서 `data`를 선언해야 함

```
version: "2.4"
services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data
volumes:
  data: {}
```

* 기본적으로 프로젝트 이름으로 볼륨을 선언하지만 `data`라는 이름으로 볼륨을 생성하고 싶으면 다음과 같이 외부로 선언을한다.

```
volumes:
  data:
    external: true
```

### 버전별 특징 및 구성 <a href="#eb-b2-84-ec-a0-84-eb-b3-84-ed-8a-b9-ec-a7-95-eb-b0-8f-ea-b5-ac-ec-84-b1" id="eb-b2-84-ec-a0-84-eb-b3-84-ed-8a-b9-ec-a7-95-eb-b0-8f-ea-b5-ac-ec-84-b1"></a>

#### Version.1 <a href="#version.1" id="version.1"></a>

[https://docs.docker.com/compose/compose-file/compose-file-v1/](https://docs.docker.com/compose/compose-file/compose-file-v1/)

* yml 문서에 버전명이 생략
* `volumes`, `networks` or `build arguments` 사용 불가
  * networking이 지원되지 않음
  * 모든 컨테이너는 기본 bridge 네트워크 에 배치 되고 해당 IP 주소의 다른 모든 컨테이너에서 연결 가능
  * 컨테이너 간 검색을 활성화 하려면 `links` 를 사용

> links\
> [https://docs.docker.com/compose/compose-file/compose-file-v1/#links](https://docs.docker.com/compose/compose-file/compose-file-v1/#links)

```
web:
  build: .
  ports:
    - "5000:5000"
  volumes:
    - .:/code
  links:
    - redis
redis:
  image: redis
```

#### Version.2 <a href="#version.2" id="version.2"></a>

[https://docs.docker.com/compose/compose-file/compose-file-v2/](https://docs.docker.com/compose/compose-file/compose-file-v2/)

* `Compose 1.6.0` 이상 `Docker Engine of version 1.10.0+.`에서 동작
* yml 문서에 버전명을 마이너 버전까지 작성 `ex) 2.1`
* 모든 서비스는 services 키 아래에 선언
* `volumes`에 Named volumes 생성 가능
* `networks`에 Network 생성 가능
* 기본적으로 모든 컨테이너는 애플리케이션 전체의 기본 네트워크에 연결되며 서비스 이름과 동일한 호스트 이름에서 검색 가능\
  이것은 링크가 거의 불필요하다는 것을 의미.

links 동작 원리\
이 컨테이너의 /etc/hosts 파일에 그 내용이 추가되어서 컨테이너에서 다른 컨테이너 들에 접근할 수 있게 된다.

```
192.168.1.186 db
192.168.1.186 database
192.168.1.187 redis
```

> 네트워크\
> [https://docs.docker.com/compose/networking/](https://docs.docker.com/compose/networking/)

Version2 예시

```
version: "2.4"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    networks:
      - front-tier
      - back-tier
  redis:
    image: redis
    volumes:
      - redis-data:/var/lib/redis
    networks:
      - back-tier
volumes:
  redis-data:
    driver: local
networks:
  front-tier:
    driver: bridge
  back-tier:
    driver: bridge
```

추가된 매개변수

* network - `aliases`
* Network - `depends_on`:

depends\_on

```
version: "2.4"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

실행시 `depends_on`에 명시된 `db`, `redis`가 뜬 뒤에 해당 서비스가 실행된다.

#### Version 2.1 <a href="#version-2.1" id="version-2.1"></a>

추가된 매개 변수

* `link_local_ips`
* build, service - `isolation`
* volumes, networks, build - `labels`
* volumes - `name`
* `userns_mode`
* `healthcheck`
* `sysctls`
* `pids_limit`
* `oom_kill_disable`
* `cpu_period`

#### Version 2.2 <a href="#version-2.2" id="version-2.2"></a>

추가된 매개 변수

* `init`
* `scale`
* `cpu_rt_runtime`, `cpu_rt_period`
* build - `network에`

#### Version 2.3 <a href="#version-2.3" id="version-2.3"></a>

추가된 매개 변수

* build - `target`, `extra_hosts`,`shm_size`
* healthcheck - `start_period`
* service - `runtime`
* volumns - `LONG SYNTAX` 제공
* `device_cgroup_rules`

LONG SYNTAX

```
 volumes:
   - type: volume
     source: mydata
     target: /data
     volume:
       nocopy: true
   - type: bind
     source: ./static
     target: /opt/app/static
```

#### Version 3 <a href="#version-3" id="version-3"></a>

[https://docs.docker.com/compose/compose-file/compose-file-v3/](https://docs.docker.com/compose/compose-file/compose-file-v3/)\
삭제된 매개 변수

* `volume_driver`
* `volumes_from`
* `cpu_shares`
* `cpu_quota`
* `cpuset`
* `mem_limit`
* `memswap_limit`
* `extends`
* `group_add`

추가된 매개 변수

* [`deploy`](https://docs.docker.com/compose/compose-file/compose-file-v3/#deploy)

#### Version 3.1 <a href="#version-3.1" id="version-3.1"></a>

추가된 매개 변수

* [`secrets`](https://docs.docker.com/compose/compose-file/compose-file-v3/#secrets)

#### Version 3.2 <a href="#version-3.2" id="version-3.2"></a>

추가된 매개 변수

* [build](https://docs.docker.com/compose/compose-file/compose-file-v3/#build) - `cache_from`
* [`ports`](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports), [`volume`](https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes) 에 `LONG SYNTAX` 지원
* network driver option - [`attachable`](https://docs.docker.com/compose/compose-file/compose-file-v3/#attachable))
* deploy - [`endpoint_mode`](https://docs.docker.com/compose/compose-file/compose-file-v3/#endpoint\_mode)
* deploy - [`placement preference`](https://docs.docker.com/compose/compose-file/compose-file-v3/#placement)

#### Version 3.3 <a href="#version-3.3" id="version-3.3"></a>

추가된 매개 변수

* [build - `labels`](https://docs.docker.com/compose/compose-file/compose-file-v3/#build)
* [`credential_spec`](https://docs.docker.com/compose/compose-file/compose-file-v3/#credential\_spec)
* [`configs`](https://docs.docker.com/compose/compose-file/compose-file-v3/#configs)

#### Version 3.4 <a href="#version-3.4" id="version-3.4"></a>

추가된 매개 변수

* build - [`target`](https://docs.docker.com/compose/compose-file/compose-file-v3/#target), [`network`](https://docs.docker.com/compose/compose-file/compose-file-v3/#network)
* [healthcheck](https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck) - `start_period`
* [update](https://docs.docker.com/compose/compose-file/compose-file-v3/#update\_config) - order
* [volumes](https://docs.docker.com/compose/compose-file/compose-file-v3/#volume-configuration-reference) - name

#### Version 3.5 <a href="#version-3.5" id="version-3.5"></a>

추가된 매개 변수

* service - [`isolation`](https://docs.docker.com/compose/compose-file/compose-file-v3/#isolation)
* networks, secrets, configs - `name`
* build- [`shm_size`](https://docs.docker.com/compose/compose-file/compose-file-v3/#shm\_size)

#### Version 3.6 <a href="#version-3.6" id="version-3.6"></a>

추가된 매개 변수

* [`tmpfs`](https://docs.docker.com/compose/compose-file/compose-file-v3/#tmpfs)
* volume mount type - `tmpfs`

#### Version 3.7 <a href="#version-3.7" id="version-3.7"></a>

추가된 매개 변수

* service - [`init`](https://docs.docker.com/compose/compose-file/compose-file-v3/#init)
* deploy - [`rollback_config`](https://docs.docker.com/compose/compose-file/compose-file-v3/#rollback\_config)
* `root of service`, `network`, `volume`, `secret`에 [extension-fields](https://docs.docker.com/compose/compose-file/compose-file-v3/#extension-fields) 제공

#### Version 3.8 <a href="#version-3.8" id="version-3.8"></a>

추가된 매개 변수

* [placement](https://docs.docker.com/compose/compose-file/compose-file-v3/#placement) - [`max_replicas_per_node`](https://docs.docker.com/compose/compose-file/compose-file-v3/#max\_replicas\_per\_node)
* [config](https://docs.docker.com/compose/compose-file/compose-file-v3/#configs-configuration-reference), [secret](https://docs.docker.com/compose/compose-file/compose-file-v3/#secrets-configuration-reference) - `template_driver` option (swarm에 배포시에만 적용)
* secret - [`driver`, `driver_opts`](https://docs.docker.com/compose/compose-file/compose-file-v3/#configs-configuration-reference) option (swarm에 배포시에만 적용)

> 참고한 문서 \
> [https://docs.microsoft.com/ko-kr/visualstudio/docker/tutorials/use-docker-compose](https://docs.microsoft.com/ko-kr/visualstudio/docker/tutorials/use-docker-compose)\
> [https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88%EC%9D%98-%EC%A3%BC%EC%9A%94-%EB%AA%85%EB%A0%B9%EC%96%B4](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88%EC%9D%98-%EC%A3%BC%EC%9A%94-%EB%AA%85%EB%A0%B9%EC%96%B4)\
> [https://docs.docker.com/compose/environment-variables/](https://docs.docker.com/compose/environment-variables/)\
> [https://docs.docker.com/compose/env-file/](https://docs.docker.com/compose/env-file/)\
> [https://docs.docker.com/compose/reference/envvars/](https://docs.docker.com/compose/reference/envvars/)
