```java
@BeforeEach  
void setUp() {  
	userRepository.deleteAll();
}
```

각각의 테스트의 독립성을 유지하고자, 이전에 실행했던 테스트의 값들을 모두 지우기 위해 deleteAll()을 많이 사용합니다.

deleteAll()은 데이터를 삭제해주는 것은 맞지만 그 역사가 지워지는 것은 아닙니다. 

#### 🤔 뭔 놈의 역사? 
예를 들어 설명해보겠습니다. JPA를 사용하는 많은 개발자들은 Entity 클래스를 만들 때 다음과 같이 만들 것이라고 생각됩니다.
```java
public class User extends BaseEntity {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "user_id")  
    private Long id;
```

이는 User가 생성될 때 마다 자동으로 ID를 증가시켜준다고 생각하면 됩니다.

중요한건 여러 테스트를 동시에 실행할 때, 이전에 User 객체를 생성하였다면 해당 ID는 없어지지 않는다는 것입니다.

따라서 고정 ID를 사용하면 내가 의도했던 User가 아닌 이미 삭제된 다른 User가 조회되어 예기치 못한 에러를 발생시킬 수 있습니다. 

