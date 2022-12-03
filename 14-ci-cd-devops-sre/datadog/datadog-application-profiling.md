# Datadog application profiling 활성화



* Datadog에서 application의 cpu, jvm 메트릭 관련을 설정 할 수 있다.
  * heapdump가 필요한 oom과 gc 튜닝
  * 스케일업 또는 스케일 아웃 시 적절한 cpu, memory 수치
  * 오래동안 또는 대량 memory 점유하는 로직

build.gradle에

`-Ddd.profiling.enabled=true`

옵션 추가

``

```kts
tasks.withType<BootBuildImage> {
    val subModuleVersion = "${rootProject.name}-${project.name}-" + rootProject.property("${rootProject.name}-${project.name}.version")
    environment = mapOf(
        "BP_DATADOG_ENABLED" to "true",
        "BPE_DELIM_JAVA_TOOL_OPTIONS" to " ",
        "BPE_APPEND_JAVA_TOOL_OPTIONS" to "-Duser.timezone=Asia/Seoul -Ddd.http.server.tag.query-string=true -Ddd.profiling.enabled=true -Ddd.version=${subModuleVersion}"
    )
}
```
