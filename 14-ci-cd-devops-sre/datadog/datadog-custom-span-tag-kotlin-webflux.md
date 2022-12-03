# Datadog custom span tag 연동(kotlin-webflux)

### 코드적용 예시(kotlin + webflux) <a href="#hardbreak-kotlin-+-webflux" id="hardbreak-kotlin-+-webflux"></a>

현재 MSA 커뮤니티 dev, prod에 올라가 있고 동작은 하지만 개선 필요한 코드 입니다.\
성능 체크도 필요.

&#x20;

1. gradle 의존성 추가

```kts
implementation("io.opentracing:opentracing-api:0.33.0")
implementation("io.opentracing:opentracing-util:0.33.0")
implementation("com.datadoghq:dd-trace-api:0.104.0")
```

\
\




2\. webFilter

```kotlin
@Component
class DatadogSpanWebFilter(private val environment: Environment) : WebFilter {
    override fun filter(exchange: ServerWebExchange, chain: WebFilterChain): Mono<Void> {
        return when {
            "dev" in environment.activeProfiles || "prod" in environment.activeProfiles -> {
                chain.filter(DatadogSpanWebExchange(exchange, environment))
            }
            else -> {
                chain.filter(exchange)
            }
        }
    }
}
```

``

3\. webExchange delegate

```kotlin
class DatadogSpanWebExchange(delegate: ServerWebExchange, environment: Environment) : ServerWebExchangeDecorator(delegate) {
    private val requestDecorator: DatadogSpanRequestDecorator = DatadogSpanRequestDecorator(delegate.request, environment)
    override fun getRequest(): ServerHttpRequest {
        return requestDecorator
    }
}
```

``

4\. webExchange request decorator

```kotlin
class DatadogSpanRequestDecorator internal constructor(delegate: ServerHttpRequest, environment: Environment) : ServerHttpRequestDecorator(delegate) {

    private val body: Flux<DataBuffer>?

    override fun getBody(): Flux<DataBuffer> {
        return body!!
    }

    init {
        val query = delegate.uri.query
        val headers = delegate.headers.toString()
        when (val span: Span = GlobalTracer.get().activeSpan()) {
            is MutableSpan -> {
                try {
                    val localRootSpan = (span as MutableSpan).localRootSpan

                    localRootSpan.setTag("custom.request.header", headers)

                    query?.let {
                        localRootSpan.setTag("custom.request.queryParams", it)
                    }
                    when {
                        delegate.method === HttpMethod.POST || delegate.method === HttpMethod.PUT -> {
                            if (MediaType.APPLICATION_JSON == delegate.headers.contentType) {
                                val bodyStream = ByteArrayOutputStream()
                                body = super.getBody().publishOn(Schedulers.boundedElastic()).doOnNext { buffer: DataBuffer ->
                                    Channels.newChannel(bodyStream).write(buffer.asByteBuffer().asReadOnlyBuffer()).run {
                                        bodyStream.flush()
                                        bodyStream.toByteArray().decodeToString().let {
                                            localRootSpan.setTag("custom.request.body", it)
                                        }
                                    }
                                }
                                bodyStream.close()
                            } else {
                                body = super.getBody()
                            }
                        }
                        else -> {
                            body = super.getBody()
                        }
                    }
                } finally {
                    MDC.remove("dd.trace_id");
                    MDC.remove("dd.span_id");
                }
            }
            else -> {
                body = super.getBody()
            }
        }
    }
}
```

``

&#x20;
