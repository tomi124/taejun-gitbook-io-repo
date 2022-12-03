# Jpa Converter 트러블 슈팅 기록



_원인_\


* 기본적으로 Value Object라는 개념을 Java에서 구현할 때는 Equals와 HashCode를 구현하여 값의 동등성을 보장하고, 값이 변경되어야 한다면, 새로운 객체로 대체할 수 있도록 구현해야 함.
* AttributeConverter 로 Entity에서 테이블 컬럼을 모델링(json to model) 컨버트를 씀
* json 컨버트 과정에서 에러 발생 시 해당 엔티티를 기존 값 -> null 로 객체 값을 바꿈
* 트랜잭션 스코프 내 서비스 로직 실행
* 영속성 스코프가 끝남. 근데 Jpa가 너 바꼈어! 값 동등성 보장 안돼. 해쉬코드랑 이퀄 체크 했는데 다르잖아. update할거야 더티체크 걸렸어 라고 함
* transaction scoper가 read slave db 여서 update는 안됨 ( 다행 > read only 잘먹네 굿 )
* 500 에러 발생

_해결방안_\


* converter의 모델을 수정함 ( 단순 string 인줄 알았으나 객체인 프로퍼티가 있었음 -> 비즈니스에서 안쓰이기 때문에 Any 처리 후 DTO에서는 null로 변경하여 변환)
* Converter에서 json 파싱에러 시 초기화 시키는게 아닌 Exception+Log로 처리 한다.
* 진행하는 프로젝트의 경우 데이터 보장이 우선적이라 했단 선택인데 결과적으로 잘못된 판단
* 에러를 회피하는게 아니라 노출 시키고 인지 해야한다.
* 에러 발생 시 명시적 logging 처리
* json 직렬화 저장의 경우 여러 케이스를 도출하여 모델링을 해봐야 함.



_\[해결에 오래 걸린 이유]_\


* Json modeling을 여러 케이스로 찾아봤는데 놓친 케이스가 있었음.
* 컨버터가 배신할 줄 몰랐음.
* json converter 사용 시 copy를 하지 않아서 동등성이 보장된다고 방심함.
* dto로 변환하기 때문에 방심함 애초에 오브젝트맵퍼 컨버터 생각못함
* 갑자기 update 컬럼이 나와서 어리둥절함.
* 개발서버는 기본적으로 Master 디비만 있기 때문에 rds host를 slave ro로 연결해도 master를 바라봐서 체크가 힘들었음.&#x20;
