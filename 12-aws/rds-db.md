# RDS DB 환경구축



RDS는 AWS에서 제공하고 있는 관계형 데이터베이스이다.\
AWS Management Console에서 RDS로 서비스로 이동할 수 있다.

### 파라미터 그룹 설정하기 <a href="#undefined" id="undefined"></a>

* 파라미터 그룹 생성 버튼을 누른다. mysql5.7을 선택하고 그룹이름은 영문으로 프로젝트이름이나 해당하는 RDS이름이라던지 자유롭게 설정한다. 설명도 동일하게 작성해준다.\
  ![](https://media.vlpt.us/images/jeongin/post/25e8c3b1-94ad-47d2-96e7-f9bcd0e4211d/image.png)
* 생성되면 편집을 하는데 해당 파라미터 그룹을 클릭하면 편집페이지로 이동이 되고 또는 해당 파라미터 그룹을 선택한 뒤 상단의 파라미터 그룹 작업을 클릭하여 편집을 선택하여 이동할 수도 있다.\
  ![](https://media.vlpt.us/images/jeongin/post/ce2103d3-8c32-4b6a-b7b2-34cc52848144/image.png)
* 파라미터를 설정해주는 것은 AWS에 설정되는 데이터베이스의 설정값을 전달하기 위한 것으로 파라미터 설정을 통해서 RDS에 데이터베이스가 생성되면 그 데이터베이스가 참조할 설정 값으로 내가 생성한 파라미터 그룹을 설정해주면 된다. 문자열을 표기하는 방식을 한글이 지원되는 데이터베이스로 만들기 위해(깨지지 않게 하기 위해) 파라미터 값을 설정한다.\
  ![](https://media.vlpt.us/images/jeongin/post/a407e969-74d4-45dd-b16e-d1d6da469162/image.png)
* 필요한 설정을 검색하면 해당하는 설정들이 검색이 된다. character\_set 설정을 utf8mb4로 해준다.(그냥 utf8은 이모티콘 저장이 안된다. mysql에서 이부분을 보안하기 위해 개량된 버전이 utf8mb4이다.) 변경사항을 저장 해준다.\
  ![](https://media.vlpt.us/images/jeongin/post/eb8973bc-0710-46ac-9e50-6f759d4014e0/image.png)
* collat 까지만 검색하여 나오는 collation\_connection은 utf8mb4\_general\_ci, collation\_server는 utf8mb4\_unicode\_ci로 설정 해준다. 변경사항을 저장한다.\
  ![](https://media.vlpt.us/images/jeongin/post/b80e28ff-519b-42fb-9438-7762da673071/image.png)\
  ![](https://media.vlpt.us/images/jeongin/post/373fa99e-602c-41a3-bfe0-cb4e4af5cca7/image.png)\
  ![](https://media.vlpt.us/images/jeongin/post/ccf1fe32-1718-41e8-b1fb-5e3227ec694a/image.png)

위와 같은 설정만 해주면 데이터베이스가 저장될 때 문자열이 깨지는 문제는 발생하지 않는다.

\


### 데이터베이스 생성 <a href="#undefined" id="undefined"></a>

아무 것도 생성이 안되어 있으면 DB 인스턴스 항목이 0개로 표시된다. 데이터베이스를 생성하려면 접속해서 바로 보이는 대시보드에서 데이터베이스 생성을 클릭해도 되고 왼쪽의 메뉴의 데이터베이스로 이동하여 데이터베이스 생성을 클릭하여 생성해도 된다.

표준생성을 선택하고 MySQL선택 버전은 기본 설정되는 버전으로 사용\
![](https://media.vlpt.us/images/jeongin/post/93f9847f-a372-4f2a-b7a8-c836c574974e/image.png)\
템플릿은 프리티어를 선택한다. 750시간 무료로 이용할수 있는 옵션이다. DB인스턴스 식별자는\
이름을 넣어준다.\
암호는 터미널에서 RDS의 mysql에 접속할 때 사용하게 되는 비밀번호이다.\
너무 단순하고 쉽게 만들면 해킹의 우려가 있으니 주의해서 생성한다.\
![](https://media.vlpt.us/images/jeongin/post/5d86239a-e068-45a5-b1a1-fdec6fbee0bd/image.png)\
프리티어를 선택했기때문에 기본적으로 설정하는 값으로 둔다.\
스토리지 자동 조정은 체크하지 않는다. 과금될 수도 있다.\
![](https://media.vlpt.us/images/jeongin/post/1ebd7850-2c99-4371-bed7-bc1c74082aca/image.png)

AWS에서 기본으로 제공하는 VPC로 자동 설정되어 있다.\
서브넷 그룹도 선택 옵션은 하나밖에 없기때문에 그대로 둔다.\
퍼블릭엑세스는 예로 선택해야 데이터베이스를 서버와 연결할 수 있다.\
기존에 생성해놓은 것이 없다면 새로 생성해야한다.\
그 외에 내용은 그림과 같이 설정해준다.\
![](https://media.vlpt.us/images/jeongin/post/894b5c8f-67c8-48df-8723-06ad1f60e25c/image.png)![](https://media.vlpt.us/images/jeongin/post/3a14f19c-7999-49d9-a15f-0cff5c877876/image.png)

추가구성에서 초기 데이터베이스의 이름은 굳이 설정해주지 않아도 된다.\
DB파라미터 그룹은 앞서 설정해놓았던 설정 값으로 선택해준다.\
자동백업 활성화는 체크하지 않는다. 역시 과금될 수도 있다.\
![](https://media.vlpt.us/images/jeongin/post/dd7c1b04-9b3d-4e92-9426-6cce2fc1981d/image.png)\
삭제 방지 활성화에 체크해놔야 실수로 디비를 지우는 일을 방지할 수 있다.\
삭제시에는 이 부분을 비활성화로 해야 디비를 삭제할 수 있다.\


![](https://media.vlpt.us/images/jeongin/post/0009c3a8-0c1b-47a1-83f1-01edce062b49/image.png)

#### 인바운드 규칙 편집 <a href="#undefined" id="undefined"></a>

처음에 데이터베이스가 생성되면 아이피주소가 내가 위치한 곳의 아이피주소로 되어있다.(집 또는 회사 등..) 이 아이피를 통해서만 데이터베이스에 접근할 수 있는데 데이터베이스는 최소한의 접근만 허용해야하기 때문에 특정 아이피 주소만을 허용하는 것이 안전하나 우리의 프로젝트시에 편의성을 위해 인바운드 규칙을 수정해준다. 작업이 완료된 후에 이 부분의 아이피 주소를 꼭 변경해놔야한다.\


![](https://media.vlpt.us/images/jeongin/post/3f4623f2-c1d9-4b5d-9633-d8e52f73f2d8/image.png)

인바운드 규칙에 접속하여 위치무관으로 변경하고 규칙을 저장한다.\


![](https://media.vlpt.us/images/jeongin/post/8fbc77f3-0bc5-4f70-9dbc-26a18b7b9e9a/image.png)

또한 아이피주소를 특정주소로 한다면 서버와 데이터베이스의 연결시에도 별도의 설정이 필요하므로 번거로워지는 단점이 있다.

이렇게 데이터베이스가 생성이 되었으면(생성되는데 몇분의 시간이 소요된다.) 로컬에서 작업한 데이터를 RDS에 있는 데이터베이스에 넣어주는 작업을 해야한다.\


![](https://media.vlpt.us/images/jeongin/post/14486a80-725a-4aa0-9a83-1118fa2d8f05/image.png)

\


### RDS의 데이터베이스로 데이터 복사하기 <a href="#rds" id="rds"></a>

```
 mysql -h RDS데이터베이스 엔드포인트 -u root -p
```

엔드포인트는 내가 생성한 데이터베이스를 클릭해서 접속하면 아래와 같이 확인할 수 있다.\
비밀번호는 데이터베이스를 설정하면서 생성한 비밀번호이다.\


![](https://media.vlpt.us/images/jeongin/post/4866e01a-2586-4d41-ae8e-5d5a64805d53/image.png)

나의 로컬 데이터베이스와 다른 점을 확인하려면 `show databases;`명령을 통해 확인할 수 있다.\
로컬에 있는 데이터베이스를 옮기위해서 데이터베이스를 생성해줘야 한다.

```
create database 데이터베이스이름 character set utf8mb4 collate utf8mb4_general_ci;
```

이렇게 생성해놓고 내 데이터베이스에서 데이터를 복사한다.

```
mysqldump -u root -p 데이터베이스명 > 데이터베이스명.sql
```

위치해 있는 폴더에 `데이터베이스명.sql`이 생성된 것을 확인할 수 있다.\
`vim 데이터베이스명.sql` 파일을 열어보면 저장된 데이터를 확인할 수 있다.

이제 이 데이터를 RDS의 데이터베이스에 넣는다.

```
mysql -h RDS데이터베이스 엔드포인트 -u root -p 데이터베이스명 < 데이터베이스명.sql
```

다시 RDS의 mysql에 접속하여 아래와 같은 명령들을 통해 데이터베이스를 확인해볼수 있다.

```
show databases;
use 데이터베이스명
show tables;
select * from 테이블명;
```
