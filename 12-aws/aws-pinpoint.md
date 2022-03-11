# AWS PINPOINT

AWS의 Amazon Pinpoint에는 Event를 저장하는 기능이 있다.

Event Stream이라 하여, Amazon Pinpoint에서 발생하는 모든 로그를 AWS Kinesis로 보내는 기능인데, 나는 보통 Elasticsearch로 보내어 로그를 관리 한다. 이후 Kibana를 통해 BI를 구성하여 쓴다.

Pinpoint EventStream 기능을 활성화 하여 Kinesis Firehose를 통해 Elasticsearch로 보내도록 구성해보자.

### Elasticsearch 도메인 생성 <a href="#elasticsearch" id="elasticsearch"></a>

![](https://media.vlpt.us/images/jakemraz/post/fa2c2d3e-2526-4c84-b0a6-853d249d3f10/Untitled.png)

새 도메인 생성 버튼 누른 후 원하는 것을 선택한다. 나는 개발 및 테스트를 선택했다.

이후 도메인 이름을 적당히 지정 후 네트워크를 구성한다. 나는 도메인 이름을 pinpoint로 지정하였다.

![](https://media.vlpt.us/images/jakemraz/post/f7666253-4222-4b9c-a25a-ed27f2ad0a0d/Untitled%201.png)

나는 네트워크 구성은 퍼블릭 액세스로 구성하였고 세분화된 액세스 제어 활성화 기능은 비활성했다.

(최소한의 보안을 위해 세분화된 액세스 제어는 활성화 하고 쓰는게 좋다.)

![](https://media.vlpt.us/images/jakemraz/post/d6910f1d-ce3d-48f7-8a1f-b2a6921063fc/Untitled%202.png)

최소한의 보안을 위해 [whatismyip.com](http://whatismyip.com) 에서 내 IP를 찾은 후 해당 IP를 추가하는 액세스 정책을 구성했다. 이렇게 해놓으면 내 IP로만 Elasticsearch에 접근이 가능하다.

이후 최종 생성까지 하고 나서 Elasticsearch 클러스터가 모두 프로비저닝 되기를 기다리자.

이는 약 10\~20분 정도 소요된다.

### Kinesis Firehose 생성 <a href="#kinesis-firehose" id="kinesis-firehose"></a>

Elasticsearch를 생성했으니 이제 Kinesis Firehose를 생성하자

Kinesis 서비스 메뉴로 들어가서 Kinesis Firehose를 생성한다.

소스는 그냥 기본 선택돼있는 것(Direct PUT or other sources) 그대로 둔다.

Transform source records와 Convert record format도 그대로 둔다.

참고로 S3에 데이터를 저장 후 AWS Glue 등을 통하여 ETL 작업 등을 수행 하려는 경우에는 데이터 포맷을 Apache Parquet 등으로 변경하여 수행 하는 것이 최적화에 도움이 된다. 하지만 지금은 Elasticsearch를 사용할 것이기 때문에 아무것도 건들지 말도록 하자.

![](https://media.vlpt.us/images/jakemraz/post/ef5527a5-ccbd-47cb-b6c0-25b53b01c3a8/Untitled%203.png)

Destination에서 Elasticsearch를 선택한다.

Domain에서는 앞서 생성한 Elasticsearch의 도메인을 선택해 준다.

그리고 나머지는 그대로 둔다.

나는 재미 삼아서 Index Rotation을 Daily로 선택했는데, 이 경우 지정한 Index 뒤에 날짜로 된 postfix가 붙는다. (예: pinpoint-2020-05-18)

나중에 용량이 꽉 차면 예전 인덱스를 지운다던지 하는 형태로 사용하기 위해 Index Rotation 기능을 사용 하는 것이 좋다.

S3 backup은 Failed records only로 두고 적당한 버켓을 하나 선택한다.

나의 경우는 Backup S3 bucket prefix는 failed\_ 로 했다.

![](https://media.vlpt.us/images/jakemraz/post/79e214bb-2814-48b5-ae79-ce10d3ff9214/Untitled%204.png)

Elasticsearch buffer conditions은 1MiB와 60seconds로 설정했다.

위와 같이 설정 할 경우 Kinesis Firehose로 전달된 데이터의 크기가 1MiB가 되거나, 60초가 지날때 마다 데이터를 Elasticsearch로 전달한다.

나머지는 다 그대로 두고 Permissions 항목에서 Create new or choose 버튼을 눌러서 Firehose → Elasticsearch 전송을 위한 새로운 Role을 생성한다.

![](https://media.vlpt.us/images/jakemraz/post/4f8b56ba-e596-41a1-adc4-599a3eedf7b3/Untitled%205.png)

이제 Create delivery stream 버튼을 눌러서 Kinesis Firehose를 생성한다. 생성까지는 약 1분정도 소요 된다.

### Pinpoint Event Stream 활성화 <a href="#pinpoint-event-stream" id="pinpoint-event-stream"></a>

Amazon Pinpoint → 원하는 프로젝트 클릭 → Settings → Event stream → Edit → Stream to Amazon Kinesis에 체크

![](https://media.vlpt.us/images/jakemraz/post/b22590b5-ef57-4534-9144-c82e0b2f72f2/Untitled%206.png)

위 화면 처럼 구성하였다.

이제 Save 버튼을 누르면,,, 드디어 Pinpoint의 모든 이벤트가 Elasticsearch에 저장이 된다.

**아직까지는 아무 캠페인도 보내지 말자. 캠페인을 보내고 난 뒤에는 type 설정이 엉퀴고 만다.**

### Timestamp 설정 <a href="#timestamp" id="timestamp"></a>

기본적으로 Pinpoint가 생성하는 Event Stream에는 arrival\_timestamp와 event\_timestamp값이 있다. 이 중 각종 이벤트를 트리거 하는데 사용되는 값은 arrival\_timestamp로 추정된다. 즉 데이터를 arrival\_timestamp 기준으로 정렬해서 볼 수 있다면 좋을 것이다.

아쉽게도 이 상태에서 Kibana를 그대로 사용하려고 하면.. 정렬 기능이 제공 되지 않기 때문에 데이터 순서가 마구잡이로 엉켜진채로 제공된다.

이를 위해 arrival\_timestamp 값이 'date' type 이라는 것을 미리 알려주자.

우선 AWS Elasticsearch 서비스로 들어가서 Kibana 옆에 있는 링크를 클릭하여 Kibana로 접속한다.

![](https://media.vlpt.us/images/jakemraz/post/970cf2df-8f55-4427-97f6-7ef17ba0750d/Untitled%207.png)

이후 왼쪽에 있는 메뉴 중 **Dev Tools**를 선택한다.

Dev Tools는 REST API를 통해 Elasticsearch에 명령을 수행 할 수 있도록 돕는다.

이제 아래의 명령을 통해 index template을 생성하자.

```json
POST _template/default_template
{
  "index_patterns" : [
      "pinpoint*"
    ],
  "order": 1,
  "mappings": {
    "properties": {
      "arrival_timestamp": {
        "type": "date",
        "format": "epoch_millis"
      }
    }
  }
}
```

**"order": 1** 에서 볼 수 있듯이 가장 우선 순위가 높게 적용 되는 index template이다.

**"template": pinpoint\*** 에서 볼 수 있듯 모든 pinpoint\* index pattern이 해당 index template의 대상이 된다.

명령어를 왼쪽칸에 입력하고 화살표 모양의 '실행' 버튼을 눌러서 REST API를 수행하자.

![](https://media.vlpt.us/images/jakemraz/post/ad748861-bee0-4d5b-8620-7c05c32c550e/Untitled%208.png)

이제 Index Pattern을 설정할 차례다.

현재는 Elasticsearch에 데이터가 아무것도 없기 때문에 Index Pattern을 설정하고 싶어도 아래와 같이 데이터가 없다는 알림만 나온다.

![](https://media.vlpt.us/images/jakemraz/post/15acf765-08dd-46d7-a1c1-6f3ca73a923f/Untitled%209.png)

이제 아무 데이터나 생성하기 위해 Pinpoint에서 아무 캠페인이나 수행하자. 나의 경우는 가장 쉽게 Email 캠페인을 수행했다.

그리고 약 1\~2분이 지나면 Pinpoint의 Event stream 기능이 Kinesis Firehose로 데이터를 보낸 후, Elasticsearch로 데이터를 전달했을 것이다.

제일 왼쪽 위 Home 버튼을 눌러서 Home으로 간 후, 화면에 보이는 버튼 중 **Use Elasticsearch data**를 클릭한다.

![](https://media.vlpt.us/images/jakemraz/post/8eb6d691-3e4d-4c4d-88de-a12c2dc15416/Untitled%2010.png)

데이터가 무사히 들어왔다면 아래와 같이 No Elasticsearch indices match your pattern에 인덱스 목록이 보여진다.

![](https://media.vlpt.us/images/jakemraz/post/264f8409-9e44-44d8-8e92-5cf1c7ce4fb3/Untitled%2011.png)

인덱스 패턴에 pinpoint\* 라고 입력 후 Next step 버튼을 클릭하자

![](https://media.vlpt.us/images/jakemraz/post/003ec265-d4aa-4a87-900a-5c60b8fb650b/Untitled%2012.png)

그러면 Time Filter field name 란에 콤보 박스가 보일 것이다.

![](https://media.vlpt.us/images/jakemraz/post/bf8b84cd-97f2-4dfb-9f97-409c7db3d361/Untitled%2013.png)

여기서 arrival\_timestamp를 선택 후 Create index pattern 버튼을 클릭하자

이후 표시되면 화면에서 arrival\_timestamp 항목이 date 타입으로 설정돼있는 것을 확인 할 수 있다.

![](https://media.vlpt.us/images/jakemraz/post/ea2919e3-acb8-4fb9-acc6-78c601c80d41/Untitled%2014.png)

이제부터 쓰고 싶은대로 Elasticsearch를 사용하자.

![](https://media.vlpt.us/images/jakemraz/post/d4df9196-86f3-46cb-b631-edb4feccbe86/Untitled%2015.png)

Discover에서 데이터가 표시 안되는 경우가 있는데, 우측 위의 Range Filter가 Last 15 minutes로 돼있는 경우에 표시가 안된다고 착각하는 경우가 많다. 원하는 타임라인으로 조정하여 사용하자.



출처 : [https://velog.io/@jakemraz/AWS-Pinpoint-Kinesis-Firehose-Elasticsearch-Event-stream-%ED%99%9C%EC%84%B1%ED%99%94](https://velog.io/@jakemraz/AWS-Pinpoint-Kinesis-Firehose-Elasticsearch-Event-stream-%ED%99%9C%EC%84%B1%ED%99%94)
