### 개요

Builder는 오브젝트 생성을 위한 디자인 패턴 중 하나입니다. 롬복이 제공하는 @Builder를 사용하면 Builder 클래스를 따로 개발하지 않고도 Builder 패턴을 사용해 오브젝트를 생성할 수 있습니다.

`User.class`

```jsx
import lombok.Builder;

@Builder
public class User {
    private final String firstName;
    private final String lastName;
    private final int age;
    private final String email;
    
    // Getters for User class...
}
```

`외부에서 Instance 생성 시`

```jsx
User user = User.builder()
                .firstName("John")
                .lastName("Doe")
                .age(30)
                .email("john.doe@example.com")
                .build();
```

### 도대체 Builder를 왜 쓰는것인가?

Builder 패턴을 사용하는 것은 생성자를 사용해 오브젝트를 만드는 것과 유사합니다. 하지만 생성자에 비해 장점이 있다면 Builder는 생성자와 다르게 매개변수의 순서를 기억할 필요가 없다는 것입니다.

### 응용 예제

```jsx
@NoArgsConstructor
@AllArgsConstructor
@Data
public class TodoDTO {
	private String id;
	private String title;
	private boolean done;

	public todoDTO(final TodoEntity entity) {
		this.id = entity.getId();
		this.title = entity.getTitle();
		this.done = entity.isDone();
	}

	public static TodoEntity toEntity(final TodoDTO dto){
		return TodoEntity.builder()
						.id(dto.getId())
						.title(dto.getTitle())
						.done(dto.isDone())
						.build();
		}
}
```