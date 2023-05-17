# 프로젝트 환경 설정 [JPA 실전(1)]

Completed?: No

### View 환경 설정

Q. Thymeleaf는 어떻게 template 패키지 안에 있는 폴더를 찾아가는 것인가?

A. 스프링 부트에서 자동으로 viewName 매핑을 해준다. 

resource:`template/` + {ViewName} + `.html`

Q. Model 객체를 써서 데이터를 넘겨 준다고 가정하자. 이 때 객체 데이터는 어떻게 넘겨주는가?

A. 객체 또한 다른 데이터 (문자열)을 넘겨주는 방식과 유사하다

```java
@GetMapping("/items")
public String example(Model model){
		Item item = new Item();
		item.setName("Banana");
		item.setPrice("2000");
		model.addAttribute("banana", item);

		return "items";
}
```

### H2 Database 설치

Database를 다운 받고 H2 폴더 내에 있는 [h2.sh](http://h2.sh) 파일을 실행해야 한다.

Q. JCBC URL 경로를 왜 설정해야 하는가?

A. 데이터 베이스 파일이 생성될 경로를 설정하는 것이다. 

JCBC URL : `jcbc:h2:~/study/jpa` 로 설정하면 `jpa.mv.db` 파일이 생성되고, 후에 JCBC 설정을 이렇게 바꿔준다. `jdbc:h2:tcp://localhost/~/study/jpa`

### JPA와 DB 설정, 동작 확인

Q. [application.properties](http://application.properties) 대신 application.yml을 사용하고 싶다. 지워도 되는가?

A. 지워도 된다.

Q. applicaion.yml에 해당 설정을 왜 해야 하는가?

A. 애플리케이션 동작에 필요한 환경을 구성하기 위해서이다. 데이터베이스 연결, JPA 설정 등의 정보를 명시함으로써 애플리케이션이 올바르게 동작할 수 있다.

Q. 해당 설정을 어디서 배우나?

A. Spring Boot Manual에 들어가서 학습을 진행해야 한다.  

Q. ddl-auto의 option은 어떤 것들이 있나?

A. 대표적으로 create, update가 있으며 이 전의 데이터베이스 정보를 유지하냐, 새로 설정하냐에 차이가 있다. 만약에 데이터베이스의 구조를 바꿨으면 update로 유지하던 설정도 create로 초기화 한 뒤 다시 실행해야 한다. 

Q. 다음 메소드에서 객체가 아닌 ID를 반환하는 이유?

A. “커맨드와 쿼리를 분리해라” 원칙에 의해, 저장을 하고 나면 리턴 값을 만들지 않는다. 

```java
public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }
```

Q. 아래 코드에서 자기 자신의 find()를 재귀 호출하는 것인가?

A. em은 EntityManager이며 em.find()는 JPA에서 제공하는 메소드이다. 

```java
public Member find(Long id) {
        return em.find(Member.class, id);
    }
```

Q. Test는 어떻게 만드는가?

A. 우선 쉬운 단축키는 다음과 같다 : `Command` + `Shift` + `T`

```java
@RunWith(SpringRunner.class) // 스프링 관련된 것을 테스트 할 거야
@SpringBootTest // Test라는 것을 스프링 프레임워크가 알도록 명시
```

Q. 오류가 떴다. 무슨 이유 일까?

![JPA application.yml error.png](..%2Fresources%2FJPA%20application.yml%20error.png)
`applicaion.yml`에 설정 정보를 잘못 입력해서 생긴 문제였다.

Q. Member 테이블이 제대로 생성되지 않는 현상

**`application.yml`** 에 제대로 설정을 안해서 발생한 현상이었다.