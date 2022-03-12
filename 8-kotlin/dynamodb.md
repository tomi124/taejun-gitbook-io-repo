# 코틀린으로 dynamoDB





kotlin(또는 java) 환경에서 dynamoDB와 소통하는 방법은 크게 2가지이다.

* dynamoDBMapper를 사용하지 않는 방식
* dynamoDBMapper를 사용하는 방식

#### DynamoDBMapper를 사용하지 않는 방식 <a href="#dynamodbmapper" id="dynamodbmapper"></a>

dbMapper를 사용하지 않는 경우, raw하게 원하는 동작에 대응하는 request 객체를 만들어서 DynamoDBClient로 request를 날려주면 된다. 특정 table로부터 원하는 item을 가져오는 getItemRequest에 대한 예시는 다음과 같다. (해당 테이블은 생성되어있음)\
`GetItem.kt`

```kotlin
//getitem : 지정해준 table과 key에 해당하는 item을 가져옴
fun main() {
    val tableName = "test_table"
    val name = "geunyoung"

    val key = HashMap<String, AttributeValue>()
    key["Name"] = AttributeValue(name)

    //request 객체 생성
    val request = GetItemRequest()
            .withKey(key)
            .withTableName(tableName)

    //dynamoDB client 생성
    val ddb: AmazonDynamoDB = AmazonDynamoDBClientBuilder.standard()
        .withRegion(Regions.AP_NORTHEAST_2)
        .build()

    try {
       // getItemRequest 전송
        val item = ddb.getItem(request).item
        if (item != null) {
            val keys: Set<String> = item.keys
            for (key in keys) {
                println("$key, ${item[key].toString()}")
            }
        } else {
            println("No item found with the key $name")
        }
    } catch (e: AmazonServiceException) {
        println(e.errorMessage)
    }
}
```

dynamoDB는 serverless한 cloud storage이기 때문에 별다른 endpoint도 없고 특별한 db connector 설정 따위도 필요없다. 그냥 client를 하나 만들어 적절한 request를 던져주면 끝!

#### DynamoDBMapper를 사용하는 방식 <a href="#dynamodbmapper" id="dynamodbmapper"></a>

위의 방식은 db 스키마가 비교적 단순할 때는 썩 괜찮아 보이지만, db 스키마가 복잡해지고 날릴 request의 요구사항이 복잡해질 경우에는 Request 객체를 생성하는 작업이 꽤 번거로워질 것이라고 예상할 수 있다. 이런 경우에는 내가 원하는 Request 객체를 손쉽게 만들어주고, db item을 kotlin 객체로 변환도 쉽게 할 수 있도록 해주는 dbMapper를 사용할 수 있다.

```kotlin
    val ddb: AmazonDynamoDB = AmazonDynamoDBClientBuilder.standard()
        .withRegion(Regions.AP_NORTHEAST_2)
        .build()
    val ddbMapper = DynamoDBMapper(ddb)
```

위와 같은 방식으로 손쉽게 DBMapper 인스턴스를 생성할 수 있다. 그리고 이 dbMapper는 @DynamoDBTable annotation의 도움을 받는 table 객체와 함께 사용하면 훨씬 다채롭게 활용이 가능하다. 아래와 같은 예시 table 객체를 생성해 주었다.\
`Event.kt`

```kotlin
@DynamoDBTable(tableName = "event")
data class Event(
    @DynamoDBHashKey(attributeName = "event_id")
    @DynamoDBTyped(DynamoDBMapperFieldModel.DynamoDBAttributeType.S)
    var event_id: String,

    @DynamoDBRangeKey(attributeName = "event_name")
    @DynamoDBTyped(DynamoDBMapperFieldModel.DynamoDBAttributeType.S)
    var event_name: String,

    @DynamoDBAttribute(attributeName = "last_login_datetime")
    @DynamoDBTyped(DynamoDBMapperFieldModel.DynamoDBAttributeType.S)
    var lastLoginDatetime: LocalDateTime? = null,

    @DynamoDBAttribute(attributeName = "last_page_view_datetime")
    @DynamoDBTyped(DynamoDBMapperFieldModel.DynamoDBAttributeType.S)
    var lastPageViewDatetime: LocalDateTime? = null,

    @DynamoDBAttribute(attributeName = "last_reservation_datetime")
    @DynamoDBTyped(DynamoDBMapperFieldModel.DynamoDBAttributeType.S)
    var lastReservationDatetime: LocalDateTime? = null
) {
    constructor() : this(event_id = "", event_name = "")
}
```

event\_id 필드를 Hash Key, event\_name 필드를 Range Key로 설정하도록 annotation을 달아줄 수 있다. 이를 가지고 실제 db table을 생성하는 코드는 아래와 같다.

```kotlin
fun createEventTable(ddb: AmazonDynamoDB, ddbMapper: DynamoDBMapper) {
    val request = ddbMapper.generateCreateTableRequest(Event::class.java)
        .withProvisionedThroughput(ProvisionedThroughput(10, 10))

    try {
        ddb.createTable(request)
        println("create event table success")
    } catch (e: AmazonServiceException) {
        println(e.errorMessage)
    }
}
```

이전에는 hash key 및 range key 혹은 LSI/GSI에 대한 정보 등을 담아 수동으로 createTableRequest를 생성해 주어야 했는데, dbMapper를 통해 db Table 객체에 대응하는 createTableRequest를 자동으로 생성할 수 있어졌다! 훨씬 코드 가독성도 높아지고, 작성하기에도 편해진 것 같다. 위 코드를 실행한 결과 테이블이 잘 생성되었음을 AWS console에서 확인할 수 있다.

![](https://media.vlpt.us/post-images/dvmflstm/0161c570-2b0f-11ea-89c4-a17182fe08af/image.png)

partition key와 sort key, RCU/WCU 등이 원하는 대로 잘 설정되어있다.\
다음은 여기에 데이터를 넣고 partition key를 이용해 scan을 해볼 것이다.\
item을 추가하는 것도 dbMapper의 save 함수를 이용하면 손쉽게 가능하다.

```kotlin
fun putItems(ddb: AmazonDynamoDB, ddbMapper: DynamoDBMapper, eventId: String) {
    val event1 = Event(event_id = eventId, event_name = "START")
    val event2 = Event(event_id = eventId, event_name = "RESERVATION")
    val event3 = Event(event_id = eventId, event_name = "LOGIN")
    val event4 = Event(event_id = eventId, event_name = "PAGE_VIEW")
    val event5 = Event(event_id = eventId, event_name = "LOGOUT")

    try {
        ddbMapper.save(event1)
        ddbMapper.save(event2)
        ddbMapper.save(event3)
        ddbMapper.save(event4)
        ddbMapper.save(event5)

        println("put items success")
    } catch (e: AmazonServiceException) {
        println(e.errorMessage)
    }
}
```

save의 parameter로 들어가는 event 객체에는 table에 대한 정보부터 partition key / sort key 등 item을 추가하는 데 필요한 모든 정보들이 들어있으므로 매우 간단히 save operation을 실행할 수 있다.\
이제 여러 데이터가 담겨 있는 이 테이블을 partition key를 이용해 scan해볼 것이다. partition key를 통한 search는 hash를 통해 이루어지므로 해당 작업은 O(1) 시간에 수행될 것으로 기대할 수 있다.

```kotlin
//partition key를 이용한 search
fun scanByPartition(ddb: AmazonDynamoDB, ddbMapper: DynamoDBMapper, eventId: String) {
    val eav = HashMap<String, AttributeValue>()
    eav[":eventId"] = AttributeValue(eventId)

    val scanExpression = DynamoDBScanExpression()
        .withFilterExpression("event_id = :eventId")
        .withExpressionAttributeValues(eav)

    val scanList = ddbMapper.scan(Event::class.java, scanExpression)

    for(item in scanList) {
        println("${item.event_id} , ${item.event_name}")
    }
}
```

partition key를 통한 scan option을 주기 위해 filter expression을 사용하였다. 이 함수의 수행 결과는 (뻔하지만) 아래와 같다.

```
3 , LOGIN
3 , LOGOUT
3 , PAGE_VIEW
3 , RESERVATION
3 , START

Process finished with exit code 0
```
