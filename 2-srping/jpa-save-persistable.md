# JPA - 엔티티 식별자 직접할당시 save 성능 최적화(Persistable)



* save의 기본동작은
* 식별자가 없으면 persist
* 식별자가 있으면 merge 이다.
* merge는 데이터베이스에서 식별자로 조회를 해본 후, 없으면 insert, 있으면 update이다.(**비효율적이다**)
* 그런데 식별자를 @GenerateValue로 자동 할당하지 않고 직접 할당하게되면 엔티티 save시, 무조건 식별자가 세팅되어 있음으로 merge를 하게된다.&#x20;
* 이 문제를 엔티티에 **Persistable** 인터페이스를 구현함으로서 자동할당시 save와 동일한 성능을 유지할 수 있다.

```
@Override 
public String getId() { 
	return id; 
} 

@Override 
public boolean isNew() {
	return createdDate == null; 
} 
```

* Persistable을 implements하게 되면 getId(), isNew() 메서드를 구현해야한다.
* 여기서 isNew에서 DB에 저장되지 않은 새로운 엔티티를 어떻게 식별할지 새롭게 재정의 하면 된다.
* 보통은 createData(등록일자)를 조합하여 등록일자가 null일 경우 새로운 엔티티로 판단하면 식별자 직접할당시 발생하는 비효율적인 성능을 개선할 수 있다.
