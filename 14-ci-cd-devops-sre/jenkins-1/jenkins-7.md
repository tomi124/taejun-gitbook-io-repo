# Jenkins 7 - 서술적 파이프라인 문법



### 기본 구조 <a href="#undefined" id="undefined"></a>

#### 1. 노드 블록 <a href="#1" id="1"></a>

* 스테이지 블록, 디렉티브, 스텝이 실행될 젠킨스 에이전트를 정의한다.

```
node('<parameter>') {<constituents>}
```

#### 2. 스테이지 블록 <a href="#2" id="2"></a>

* 스테이지 블록은 같은 목적을 가진 스텝과 디렉티브의 모음이다.

```
stage('<parameter>') {<constituents>}
```

#### 3. 디렉티브 <a href="#3" id="3"></a>

* 디렉티브의 가장 큰 목적은 환경 변수, 옵션, 파라미터, 트리거, 툴을 제공해 노드 블록과 스테이지 블록, 스텝을 지원하는 것이다.

#### 4. 스텝 <a href="#4" id="4"></a>

* 스텝은 서술적 파이프라인을 구성하는 가장 중요한 요소
* 젠킨스에서 무엇을 할지 명령을 내리는 것

```
node('master') {
    // Directive 1
    def mvnHome
    
    // Stage block 1
    stage('Preparation') { 
        // Step 1
        git 'https://github.com/jglick/simple-maven-project-with-tests.git'
        // Directive 2
        mvnHome = tool 'M3'
    }
    
    // Staage block 2
    stage('Build') {
        // Step 2
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
            }
        }
    }
    
    // Stage block 3
    stage('Results') {
        // Step 3
        junit '**/target/surefire-reports/TEST-*.xml'
        // Step 4
        archiveArtifacts 'target/*.jar'
    }
}
```

* any를 파라미터로 사용하면 모든 스테이지 노드와 스텝, 디렉티브는 임의의 젠킨스 슬레이브 중 하나에서 수행된다.

출처 : [https://velog.io/@jae\_cheol/%EC%84%9C%EC%88%A0%EC%A0%81-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EB%AC%B8%EB%B2%95](https://velog.io/@jae\_cheol/%EC%84%9C%EC%88%A0%EC%A0%81-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EB%AC%B8%EB%B2%95)
