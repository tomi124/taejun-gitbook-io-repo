# MongoDB



### SQL vs NoSQL <a href="#sql-vs-nosql" id="sql-vs-nosql"></a>

![](https://media.vlpt.us/images/ckstn0777/post/9451e719-3f98-4124-9a29-67ea96925ac0/image.png)

#### SQL <a href="#sql" id="sql"></a>

SQL(Structured Query Language)은 관계형 데이터베이스 관리 시스템(RDBMS)의 데이터를 관리하기 위해 설계된 특수 목적의 프로그래밍 언어이다. 많은 수의 데이터베이스 관련 프로그램들이 SQL을 표준으로 채택하고 있다. - 위키피디아 -

대중적으로 가장 많이 사용되는 Oracle, MySQL 등이 이 범주에 포함된다.

#### NoSQL <a href="#nosql" id="nosql"></a>

NoSQL(원래 의미: non SQL 또는 non relational) 데이터베이스는 전통적인 관계형 데이터베이스 보다 덜 제한적인 일관성 모델을 이용하는 데이터의 저장 및 검색을 위한 매커니즘을 제공한다. NoSQL 데이터베이스는 빅데이터와 실시간 웹 애플리케이션의 상업적 이용에 널리 쓰인다. SQL 계열 쿼리 언어를 사용할 수 있다는 사실을 강조한다는 면에서 "Not only SQL"로 불리기도 한다.

NoSQL 분류는 다음과 같다.

* Wide Columnar Store : 카산드라
* Document Store : MongoDB
* Key-Value Store : 다이나모, 레디스
* Graph Store: Neo4j

그 중에서 유명한건 카산드라, MongoDB, AWS의 DynamoDB, Redis 정도?

#### SQL과 NoSQL 비교 <a href="#sql-nosql" id="sql-nosql"></a>

![](https://media.vlpt.us/images/ckstn0777/post/2c6d8bce-aa94-4f68-90aa-8f5c04d49ddf/image.png)

👉 출처 : [https://ko.wikipedia.org/wiki/NoSQL](https://ko.wikipedia.org/wiki/NoSQL)

성능면과 확장성 면에서는 NoSQL이 SQL보다 우수하다. 또한 유연하며 복잡성이 낮은것이 특징이다. 하지만 마냥 좋은점만 있는건 아니다. **ACID 트랜잭션**(원자성/일관성/고립성/영구성)을 보장받기 위해서는 RDBMS를 쓰는 편이 좋다. 가령 은행업무나 회사업무같은 중요한 DB는 RDBMS를 쓰는 것을 권장한다.

> ✅ 결론적으로 SQL, NoSQL 뭐가 좋고 나쁘냐가 아니라 사용하는 용도에 따라 선택하면 된다.

\


### MongoDB 알아보기 <a href="#mongodb" id="mongodb"></a>

MongoDB는 C++로 작성된 오픈소스 **문서지향(Document-Oriented)** 크로스 플랫폼 데이터베이스이다.

#### 1. Document <a href="#1-document" id="1-document"></a>

Document는 RDBMS에서의 Row(혹은 튜플)과 동일한 개념. 예를 들어 아래와 같은 JSON 형태의 key-value 쌍으로 이루어진 데이터 구조를 하나의 Document라고 보면 된다.

```sql
{
    "_id": "5f2ad6b54866e5109dd2367b"
    "username": "홍길동",
    "hashedPassword": "비밀번호",
}
```

각 Document는 **\_id**를 갖고 있는데 이 값은 유일하다. Primary Key랑 동일한 개념.

특이하게도 RDBMS처럼 스키마로 정해진 뭔가가 없다. 따라서 username, hashedPassword 밑에다가 email을 추가한 Document를 새로 생성해도 문제 없이 동작한다.

#### 2. Collection <a href="#2-collection" id="2-collection"></a>

Collection은 Document의 그룹이다. RDBMS로 따지자면 Table과 비슷한 개념. 다만 위에서 말했듯이 스키마를 가지고 있지 않다.

#### 3. Database <a href="#3-database" id="3-database"></a>

Database는 Collection들의 물리적인 컨테이너이자 가장 상위 개념이다. RDBMS에서의 Database와 동일하다.

#### Q. MongoDB에서 Join은? <a href="#q-mongodb-join" id="q-mongodb-join"></a>

RDBMS에서는 항상 Join이 일상적으로 많이 사용한다. 그렇다면 MongoDB도 조인이란 개념이 있을까? 불가능하지는 않지만 일반적으로는 그렇게 쓰이지 않는다.

post와 comment가 있다고 가정해볼까? post하나에는 comment가 여러개가 있으니까 1대다 관계이다. 따라서 RDBMS를 썼다면 2개의 테이블로 분리시켰을거다. 하지만 MongoDB는 분리시키지 않는 대신에 post 내부에 comment를 넣는다. 이를 **서브다큐먼트**라고 한다.

따라서 아래와 같은 형태가 나오게 됩니다.

```javascript
{
   _id: POST_ID,
   title: POST_TITLE,
   content: POST_CONTENT,
   username: POST_WRITER,
   tags: [ TAG1, TAG2, TAG3 ],
   comments: [
     { 
        username: COMMENT_WRITER,
        mesage: COMMENT_MESSAGE,
        time: COMMENT_TIME
     },
   ]
}

출처 : https://velopert.com/436
```

\


### 참고 <a href="#undefined" id="undefined"></a>

* poiemaweb 사이트 : [https://poiemaweb.com/mongdb-basics](https://poiemaweb.com/mongdb-basics)
* velopert님 블로그 : [https://velopert.com/436](https://velopert.com/436)
* SQL vs NoSQL : [https://velog.io/@thms200/SQL-vs-NoSQL](https://velog.io/@thms200/SQL-vs-NoSQL)
