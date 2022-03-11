# 빌드 개념 정리

### 빌드? <a href="#eb-b9-8c-eb-93-9c-3f" id="eb-b9-8c-eb-93-9c-3f"></a>

> In [software development](https://en.wikipedia.org/wiki/Software\_development), a **build** is the process of converting [source code](https://en.wikipedia.org/wiki/Source\_code) files into standalone [software artifact(s)](https://en.wikipedia.org/wiki/Artifact\_\(software\_development\)) that can be run on a computer, or the result of doing so. (출처 : 위키피디아 [software build](https://en.wikipedia.org/wiki/Software\_build))

위의 정의에서도 알 수 있듯이, '빌드'는 소스 코드 파일을 각각의 컴퓨터에서 독립적으로 실행할 수 있는 소프트웨어 산출물로 변환하는 과정, 또는 그 과정에 따른 산출물을 가리킨다.

JAVA의 경우에는 빌드 과정에서 개발자가 작성한 소스코드(\*.java)가 JVM이 이해할 수 있는 바이트코드(\*.class)로 **컴파일**되고, 각종 설정 및 외부 라이브러리 등과 함께 **패키징되어** 즉시 **실행**할 수 있는 상태의 파일이 생성된다.

### JAR? WAR? EAR? <a href="#jar-3f-war-3f-ear-3f" id="jar-3f-war-3f-ear-3f"></a>

JAVA의 경우 빌드 결과 생성되는 파일은 **JAR, WAR, EAR**로 세 종류가 있다. WAR나 EAR의 경우 zip 포맷을 따르는 표준적인 JAR 파일이다. 즉 JAR, WAR, EAR 모두 확장자만 다를 뿐(\*.jar, \*.war, \*.ear) 압축 포맷 자체는 동일하다. 하지만 세 종류의 파일은 그 내부에 담고 있는 구성요소가 다르다. JAR보다는 WAR가, WAR보다는 EAR가 보다 많은 구성요소를 포함하는데, 그림으로 나타내면 아래와 같다. 보다 자세하게 살펴보도록 하자.

![jar,war,ear](https://i.stack.imgur.com/ZLdF7.png)

\
\
(이미지 출처 : [Difference between jar and war in Java (stackoverflow)](https://stackoverflow.com/questions/5871053/difference-between-jar-and-war-in-java)

#### JAR <a href="#jar" id="jar"></a>

JAR는 **J**ava **AR**chive의 약자이다. 이름에서도 알 수 있듯이, 빌드에 의해 생성되는 가장 기본적인 자바 아카이브 파일 포맷이다. JAR는 자바 클래스(\*.class), 메타데이터를 담고 있는 menifest 파일, 그리고 그 외에 필요한 이미지, 텍스트 등을 담고 있다. menifest 파일은 애플리케이션 실행의 시작 지점이 되는 public static void main 함수가 포함된 함수의 위치, 애플리케이션 실행 과정에서 의존성이 존재하는 클래스들의 경로, 버젼 정보 등을 담고 있다. 결과적으로 `java -jar myjar.jar`와 같은 명령이 실행되면 menifest 파일에 포함된 정보들을 바탕으로 코드의 진입지점 및 의존성이 파악되고, 그에 따라 애플리케이션이 정상적으로 실행된다.

JAR 파일은 독립적인 실행이 가능한 만큼 다른 자바 애플리케이션의 모듈 또는 라이브러리로 활용될 수 있다. 외부 라이브러리가 저장되는 경로를 살펴보면 jar 확장자를 가지는 파일들을 다수 발견할 수 있다.\
![](https://user-images.githubusercontent.com/67428295/107145722-d0876380-6986-11eb-966d-4b9a285a5f56.png)\
\


#### WAR <a href="#war" id="war"></a>

WAR는 **W**eb **A**pplication **R**esource 또는 **W**eb Application **AR**chive의 약자이다. 이름에서도 알 수 있듯이, WAR는 웹 어플리케이션 배포를 위한 파일 포맷이다. JAR과는 달리 WAR는 자바빈 클래스 외에도 웹 애플리케이션 구동을 위한 추가적 파일들을 담고 있다. 즉 자바 서블릿 클래스(\*.class), JSP(\*.jsp)와 같이 웹 기반 요청 처리를 위한 자원, html과 같은 정적 자원, 그리고 서블릿의 URL 맵핑 등을 관리하는 web.xml과 같은 설정 파일을 포함한다. 또한 추가적으로 라이브러리 목적으로 사용되는 JAR 파일을 포함할 수 있다.

JAR와는 달리 WAR의 경우에는 톰캣과 같은 웹 어플리케이션 서버 위에서만 동작한다. 자바 웹 어플리케이션 실행을 위해서는 JSP와 서블릿의 처리가 필요하고, 이러한 처리를 위해서는 서블릿 처리가 가능한 웹 어플리케션 서버가 필요하기 때문이다.

#### EAR <a href="#ear" id="ear"></a>

EAR는 **E**nterprise **A**pplication a**R**chive의 약자이다. EAR는 여러 개의 JAR 및 WAR들을 모듈로 포함한다. EAR을 활용하면, 여러 모듈들을 한번에 웹 어플리케이션 서버에 배포할 수 있다.\
~~사실 당장은 어떻게 돌아가는 것인지 잘 모르겠다. 그러니 여기까지만...~~\
\
\


### 스프링부트는 JAR로도 배포하던데? <a href="#ec-8a-a4-ed-94-84-eb-a7-81-eb-b6-80-ed-8a-b8-eb-8a-94-jar-eb-a1-9c-eb-8f-84-eb-b0-b0-ed-8f-ac-ed-95" id="ec-8a-a4-ed-94-84-eb-a7-81-eb-b6-80-ed-8a-b8-eb-8a-94-jar-eb-a1-9c-eb-8f-84-eb-b0-b0-ed-8f-ac-ed-95"></a>

최근에는 스프링부트를 중심으로 학습을 진행하고 있는데, 스프링부트 프로젝트의 경우에는 웹 자원을 포함함에도 불구하고 JAR 파일로 빌드해서 배포할 수 있다. 앞서 언급했듯이 웹 자원을 포함하는 웹 어플리케이션의 경우에는 WAR로 배포하는 것이 정상이다. 그런데 왜 WAR이 아니라 JAR로의 빌드가 가능한 것일까?\
![](https://user-images.githubusercontent.com/67428295/107144836-26590d00-6981-11eb-91ca-3ff919969ad2.png)

**스프링부트는 웹 어플리케이션 서버를 내장하고 있기 때문이다.** 즉 굳이 서버를 설치하고 그 위에서 WAR 파일을 작동시킬 필요가 없다. JAR 파일을 실행하면 스프링부트에 내장된 서버(tomcat(dafault), jetty, undertow 중 선택)가 자동적으로 실행되고, 그 위에서 자바 웹 어플리케이션이 구동된다.
