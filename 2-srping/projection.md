---
description: Projection
---

# Projection

### &#x20;<a href="#projection-basic" id="projection-basic"></a>

### Projection <a href="#projection" id="projection"></a>

엔티티의 속성들이 너무 많을 때, 일부 데이터만 가져오는 방법이다.

아래와 같이 comment 엔티티가 있다고 하자\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2F003609c0-43a7-48b4-8a5f-f10f9eef4539%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.33.55.png)

#### 인터페이스 기반 <a href="#undefined" id="undefined"></a>

**closed projection**

위의 정보들 중 comment, up, down만 관심있다고 가정하여, 임의로 인터페이스를 만들어준다.

![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2F46d3f701-fe09-4c2e-b0a4-c86f6bae6371%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.35.29.png)

어떤 Post안에 있는 comment를 조회한다.(findByPost\_Id)

만들어준 인터페이스 타입으로 쿼리를 만들어주고 실행시켜보면\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2Faad7e428-8200-4715-8058-c976fd9c8d10%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.38.31.png)\
comment, up, down 값만 select 해오는 것을 볼 수 있다.\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2Fa292b759-9927-4dd7-86c4-b94b0afca2f4%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.40.16.png)

**open projection**

open projection은 모든 요소를 다 가져와서 두 개의 요소를 합쳐서 보여줄 수 있다. 따라서 성능최적화는 불가능하다.

![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2Fa17758dc-84aa-406b-98c9-1040e23de423%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.47.14.png)

#### 혼합 <a href="#undefined" id="undefined"></a>

필요한 요소만 가져와서 커스텀하게 요소를 다룰 수도 있다.\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2Fff59141a-4b1c-4f78-9893-25cc09837ab8%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.50.02.png)

#### 클래스 기반 <a href="#undefined" id="undefined"></a>

클래스 기반으로도 프로젝션 처리가 가능하다.\
DTO를 생각하면 된다.

![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2F87af22ec-6666-497c-bef9-dde4377ff9a3%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.51.53.png)

위의 인터페이스와 아래의 클래스는 같은 기능을 한다.

![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2Fee924855-a293-4ee8-9ad9-a17792c729b0%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.53.22.png)

#### Tip <a href="#tip" id="tip"></a>

프로젝션을 여러개 만들 수도 있다.\
현재 post\_id 값으로 CommentSummary 타입을 가져오는 쿼리를 만들었는데, post\_id 값으로 다른 값도 같이 가져올 수 있다.

예를 들어 아래와 같이 comment만 가져오는 인터페이스도 있다고 하자.\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2F0c7115e5-c7d5-49ab-9945-d9fd913cf1fe%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.57.07.png)

post\_id값으로 값을 가져오고 싶지만, 메서드 명이 같아서 선언해줄 수가 없다.\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2F24a16d58-8fd1-4725-8ad5-18c2b7e8acbd%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.58.03.png)

이런 경우 Generic 타입을 사용하면 된다.\
![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2F4aeb810d-6f43-4a4e-973a-941da6582c48%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%207.59.49.png)

![](https://velog.velcdn.com/images%2Fmax9106%2Fpost%2Fe2b7218b-df31-480e-8cc1-4077b19d998a%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-04-01%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%208.01.14.png)
