# graphQl



## 1. 개발배경 <a href="#1" id="1"></a>

> **페이스북이 개발한 Query Language**

* Query 란 데이터베이스에 정보를 요청하는 것으로, Query Language는 **데이터베이스에 접근할 수 있는 언어** 라고 이해할 수 있다.

GraphQL은 REST API의 단점을 해결하기 위해 개발되었는데, 가장 큰 장점으로 **Endpoint가 "하나" 라는 점이다.**

Rest API는 정의된 여러 Endpoint에 액세스하여 데이터를 수집해야 한다.\
**클라이언트에서 필요한 데이터는 언제든 변화할 수 있는데, Rest API는 고정되어 있는 데이터를 수집하기 때문에 필요이상의 데이터를 받거나(Overfetching), 필요한 정보가 없는 경우가 발생할 수 있다.(Underfetching)**

Endpoint를 하나로 설정하여 데이터베이스를 구축한다면, 여러 Endpoint에서 데이터를 구축하지 않아도 되기 때문에 유연하게 대처할 수 있다.

## 2. GraphQl 서버언어와 클라이언트 종류 <a href="#2-graphql" id="2-graphql"></a>

#### 서버언어 종류 <a href="#undefined" id="undefined"></a>

* 자바스크립트
* 파이썬
* 루비
* 자바
* C#
* 스칼라
* Go
* PHP

#### 클라이언트 종류 <a href="#undefined" id="undefined"></a>

1. Apollo
2. Relay

## 3. GraphQL 의 효율성 <a href="#3-graphql" id="3-graphql"></a>

#### 1. URL, View를 정의할 필요가 없다. <a href="#1-url-view" id="1-url-view"></a>

* API를 만드는 시간이 매우 절약된다.

#### 2. 프론트엔드에서 fetch로 API를 가져오지 않아도 된다. <a href="#2-fetch-api" id="2-fetch-api"></a>

* 프론트엔드 쪽에서 Fetch로 API를 가져와 JSON, Response등을 작업하지 않아도 된다.
* 대표적으로 Apollo를 활용하면, fetch, JSON 작업을 모두 지원하여 준다.

#### 3. React와 함께 쓰면 매우 높은 효율을 가져올 수 있다. <a href="#3-react" id="3-react"></a>

* End-Point가 하나 이고, 많은 부분을 셋팅해주기 때문이다
* React 또한 페이스북이 개발했기 때문에, React와 호환하는 다양한 라이브러리가 존재하고, 커뮤니티 지원도 있다.

#### 4. 리덕스를 대체할 수 있다! <a href="#4" id="4"></a>

(더 간단하게 리덕스를 대체할 수 있다는 것이 신기했다.)

> GraphQL + Apollo 조합의 효율

* GraphQL로 Local state를 관리할 수 있다.
