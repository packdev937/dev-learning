의존성 주입은 다음과 같이 진행합니다.
```kotlin
@SpringBootTest
class UserServiceTest (
	@Autowired private val userRepository: UserRepository,
	@Autowired private val userService :UserService
)
```

이 때 중복되는 @Autowired는 다음과 같이 빼줄 수 있습니다.

``` kotlin
@SpringBootTest
class UserServiceTest @Autowired constructor (
	private val userRepository: UserRepository,
	private val userService :UserService
)
```

