---
description: graphql을 알아보자
---

# 11) GRAPHQL



\


![](https://i0.wp.com/hanamon.kr/wp-content/uploads/2021/05/GraphQL.png?fit=1140%2C474\&ssl=1)

### **❓GraphQL이란?**

GraphQL(이하 GQL)은 Facebook에서 만든 Graph Query Language로 **어플리케이션 레이어 쿼리 언어**이다.

GQL은 Structed Qurery Language(이하 SQL)와 마찬가지로 쿼리 언어이다.

SQL은 데이터베이스 시스템에 저장된 데이터를 효율적으로 가져오는 것이 목적이고,\
GQL은 웹 클라이언트가 데이터를 서버로 부터 효율적으로 가져오는 것이 목적이다.

SQL의 문장(statement)은 주로 백엔드 시스템에서 작성하고 호출하는 반면, GQL의 문장은 주로 클라이언트 시스템에서 작성하고 호출한다.

GQL즉, **API를 위한 쿼리 언어**이며, **타입 시스템**을 사용하여 쿼리를 실행하는 **서버사이드 런타임**이다.

GraphQL은 특정한 **데이터베이스**나 **스토리지**에 귀속되어 있지 않으며, **기존 코드와 데이터에 의해 대체**된다.

**SQL 쿼리 예시**

`SELECT plot_id, species_id, sex, weight, ROUND(weight / 1000.0, 2) FROM surveys;`

**GQL 쿼리 예시**

`{hero {namefriends {name}}}`

**서버사이드 GQL 어플리케이션**은 GQL로 작성된 쿼리를 입력으로 받아 쿼리를 처리한 결과를 다시 클라이언트로 돌려준다.

HTTP API 자체가 특정 데이터베이스나 플렛폼에 종속적이지 않은 것 처럼 마찬가지로 GQL 역시 어떠한 특정 데이터베이스나 플렛폼에 종속적이지 않다.

(심지어 네트워크 방식에도 종속적이지 않다. 일반적으로 GQL의 인터페이스간 송수신은 네트워크 레이어 L7의 HTTP POST 메서드와 웹소켓 프로토콜을 활용한다. 필요에 따라서는 얼마든지 L4의 TCP/UDP를 활용하거나 심지어 L2 형식의 이더넷 프레임을 활용 할 수도 있다.)

&#x20;

***

&#x20;

### **💦 REST API의 한계**

REST API의 개념을 간단하게 말하자면 모든 Resource들을 하나의 Endpoint에 연결하고 연결된 Endpoint는 Resource와 관련된 내용만 관리하게 하는 것이다.

이는 단순한 서비스에서는 아주 좋지만 복잡한 서비스나 클라이언트의 요청사항에 따라 **Over-Fetching**과 **Under-Fetching**이 발생한다.

또한 REST API로 여러 환경에서 필요한 정보들을 Resource 별로 Endpoint를 갖도록 구현하는 것은 어렵다. 한마디로 비슷하지만 Endpoint가 다른 API가 많이 파생된다.

#### **# Over-Fetching**

예를 들어, 사용자의 데이터를 조회하는 /user/API가 있다고 가정한다.

이 때, 사용자 번호: 1 에 해당하는 데이터를 조회한다면 아래와 같은 형태가 된다.

```
GET /user/1/
response body
{
"user_no":1,
"user_name": "test",
"user_grade": "VVIP",
"zip": "11053",
"last_login_timetamp": "2019-08-08 12:11:44",
...
}
```

여기에서 클라이언트는 1번에 해당하는 유저의 이름만을 사용하고자 한다고 해도 유저 이름만 반환하는 API가 없다면 위와 같은 /user/1/API를 호출한 다음, user\_name을 가져와 사용해야한다. 이 때, user\_grade, zip 등등의 데이터는 사용하지 않는 데이터도 같이 반환받는다. 이는 곧 리소스의 낭비라고 볼 수 있고 이와 같은 낭비를 Over-Fetching이라 명한다.

#### **# Under-Fetching**

쇼핑몰 서비스의 경우, 로그인한 사용자의 장바구니 정보를 보여준다고 가정하면 여러 API를 호출하게 된다.

`/user/1/`

`/cart/`

`/notification/`

`/wish/...`

요청에 맞게 유효한 데이터를 보여주기 위해 여러 API를 호출하게 되는 경우를 Under-Fetching이라 한다.

#### **# REST API 목록**

REST API 방법론으로 API를 만든다고 하면 Resource 별로 Endpoint를 갖기 때문에 비슷하지만 다른 API를 생성하게 된다.

`/item/`

`/item/detail/`

`/item/image/`

`/item/notice/`

`/item/manage/...`

이는 서비스가 커져갈 때 관리 포인트가 늘어나게 되므로 개발자나 클라이언트에게 부담이 된다.

&#x20;

***

&#x20;

### **💡 GraphQL**

GraphQL은 위에서 설명한 것처럼 REST API의 한계를 극복하고자 나왔다.

Endpoint는 통상 1개만 생성하고 클라이언트에게 필요한 데이터는 클라이언트가 직접 쿼리를 작성, 호출하여 반환 받는다.

> **위의 Under-Fetching 문제를 GraphQL을 사용하면 손쉽게 해결된다.**

**요청 쿼리**

query{user(user\_no:1){user\_name\}}

**반환 데이터**

{"data": {"user": {"user\_name": "jim",\}}}

> **또한 Over-Fetching 문제도 GraphQL을 사용해 해결할 수 있다.**

**요청 쿼리**

query{cart{product\_nameprice}notification{is\_read}user(user\_id:1){user\_nameuser\_grade\}}

**반환 데이터**

{"cart": \[{"product\_name": "shoes","price": 12000}, ...],"notifications": \[{is\_read: true}],"user": {"user\_name": "jim","user\_grade": "VVIP"\}}

#### **👍 장점**

* 클라이언트가 필요한 데이터만 반환할 수 있다.
* 1번의 호출로 원하는 데이터를 한번에 가져올 수 있다.
  * REST API의 N+1 Problem을 해결 할 수 있다.
* 확장이 용이

#### **👎 단점**

* 백엔드, 클라이언트 개발자 양쪽 다 **러닝커브**가 있다.
* 단순한 서비스에서는 사용하기가 복잡하다.
  * 대부분의 언어에서 라이브러리로 제공한다.
* 캐싱 기능의 구현이 복잡하다.
* 요청이 text로 날아가기 때문에 File 전송 등을 구현하기가 어렵다.
