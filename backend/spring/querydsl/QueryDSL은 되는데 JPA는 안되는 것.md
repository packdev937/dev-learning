그건 바로 런타임에 발생할 수 있는 예외 처리를 막는 것입니다. 

한 예제를 가져와봤는데요,
불리안 변수로 책의 반납 상태를 확인하던 서비스가 있었습니다. 

```kotlin
	fun findByBookNameAndStatus(bookName: String, isReturn: Boolean): UserLoanHistory?
```

서비스의 유연함을 더하기 위해 개발자는 isReturn 필드를 UserLoanStatus Enum 클래스로 변경했습니다. 하지만 매개변수로 넘겨주다 보니 이는 추적이 되지 않습니다. 

당연하게 런타임 예외가 발생하게 됩니다. 

만약 QueryDSL로 위를 구현했다면 컴파일 타임에 이를 해결할 수 있을 것입니다. 

따라서 이는 JPA의 단점이자 QueryDSL의 장점이라고 볼 수 있습니다. 