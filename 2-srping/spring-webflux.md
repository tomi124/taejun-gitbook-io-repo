# Spring Webflux 예외처리

기존 WebMvc에서는 `ControllerAdvice` 와 `ExceptionHandler` 어노테이션으로\
Global한 예외처리를 하였는데\
WebFlux에서 Functional Endpoint 로 구현을 하였다면 다른 방식으로 예외처리를 해야합니다.

### WebExceptionHandler <a href="#webexceptionhandler" id="webexceptionhandler"></a>

구글링을 해본 결과 Functional Endpoint 방식같은 경우는 WebExceptionHandler 으로 처리를 해주면 된다고 합니다.

```kotlin
@Component
@Order(-2)  // Order(-1) 에 등록된 DefaultErrorWebExceptionHandler 보다 높은 우선순위를 부여하기 위함.
class GlobalExceptionHandler: WebExceptionHandler {
    override fun handle(exchange: ServerWebExchange, ex: Throwable): Mono<Void> =
        handle(ex)
            .flatMap {
                it.writeTo(exchange, HandlerStrategiesResponseContext(HandlerStrategies.withDefaults()))
            }.flatMap {
                Mono.empty<Void>()
            }

    fun handle(throwable: Throwable): Mono<ServerResponse> {
        return when (throwable) {
            is ApiException -> {
                createResponse(throwable.exception)
            }
            else -> {
                createResponse(ExceptionMessage.INTERNAL_SERVER_ERROR)
            }
        }
    }

    private fun createResponse(exception: ExceptionMessage): Mono<ServerResponse> {
    	// bodyValue에는 저희가 원하는 Response 형태를 넣어 줍시다.  
        return ServerResponse.status(exception.statusCode).bodyValue(Response(exception.message, null))
    }
}

private class HandlerStrategiesResponseContext(val strategies: HandlerStrategies) : ServerResponse.Context {
    override fun messageWriters(): List<HttpMessageWriter<*>> {
        return this.strategies.messageWriters()
    }

    override fun viewResolvers(): List<ViewResolver> {
        return this.strategies.viewResolvers()
    }
}
```

```kotlin
class ApiException(
    val exception: ExceptionMessage
): RuntimeException()
```

```kotlin
enum class ExceptionMessage(
    val statusCode: HttpStatus,
    val message: String
) {
    TYPE_EXCEPTION(HttpStatus.BAD_REQUEST, "타입이 잘못되었습니다."),
    NOT_FOUND(HttpStatus.NOT_FOUND, "해당 데이터를 못 찾았습니다."),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "알 수 없는 서버 에러입니다.")
}
```

`@Order(-2)` 어노테이션을 넣은 이유는\
만약 넣지 않게되면 예외가 발생 시 `DefaultErrorWebExceptionHandler` 가 발생하기때문에 `GlobalExceptionHandler`가 작동하지 않는 상황이 발생하기 때문에 우선순위를 -2로 설정을 해줬습니다.\
`DefaultErrorWebExceptionHandler`의 우선순위는 -1

예외가 발생하면 Throwable의 Exception의 종류에 따라 Response를 만들면 되는데\
하지만 저는 Exception의 종류는 한가지로 고정시키고 Exception 프로퍼티로 예외 내용을 가지고 있게끔 만들어줬습니다. (Exception 종류별로 만들어서 따로 처리해줘도 됩니다.)

`HandlerStrategiesResponseContext`에서는 생성한 `Mono<ServerResponse>`를 실제 Response에 설정을 하여 저희가 원하는 형태로 전달에 성공합니다.

#### 예외 발생 시키기 <a href="#undefined" id="undefined"></a>

```kotlin
// Post Handler
fun getHotPost(request: ServerRequest): Mono<ServerResponse> {
	val postType = request.pathVariable("postType")

	// 인기 게시물 타입
	if(postType !== "hot")
		throw ApiException(ExceptionMessage.TYPE_EXCEPTION)
	....
    
}
```

`RuntimeException`을 상속받은 Exception 종류를 throw 해주면 됩니다.
