# Datadog Log 한글이 깨지는 경우

### Spring Boot 프로젝트의 경우 <a href="#spring-boot" id="spring-boot"></a>

BootBuildImage task 에 `-Dfile.encoding=UTF-8` 옵션을 추가합니다.

&#x20;\> build.gradle.kts



#### timezone도 같이 <a href="#timezone" id="timezone"></a>

`-Duser.timezone=Asia/Seoul`
