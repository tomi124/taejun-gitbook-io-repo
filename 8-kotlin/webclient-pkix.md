# 코틀린 Webclient PKIX 이슈?

개발 중 API나 카프카 컨슈머에서 를 Webclient를 통한 통신 시 아래의 에러가 발생했다!

```
failMessage" : "PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

아래의 TODO를 확인해보자!



```
fun webClient(webclientBuilder: WebClient.Builder): WebClient {
        // TODO: SSL 확인 안하는 설정
        val sslContext = SslContextBuilder
            .forClient()
            .trustManager(InsecureTrustManagerFactory.INSTANCE)
            .build()

        return HttpClient.create()
            .secure { t -> t.sslContext(sslContext) }
            .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, props.connectionTimeout.toMillis().toInt())
            .doOnConnected {
                it.addHandler(ReadTimeoutHandler(props.readTimeout.toMillis(), TimeUnit.MILLISECONDS))
                it.addHandler(WriteTimeoutHandler(props.writeTimeout.toMillis(), TimeUnit.MILLISECONDS))
            }.let {
                webclientBuilder.clientConnector(ReactorClientHttpConnector(it)).build()
            }
    }
```

출처 : jongwoo-lee
