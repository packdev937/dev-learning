## 1. 개요 
MSA 기반의 헥사고날 아키텍쳐 프로젝트를 진행하고 있습니다. 작성한 코드의 흐름은 대부분 다음과 같습니다. 
`Controller -> ApplicationService -> ApplicationHandler -> DomainService  

문득 위와 같은 구조가 지닌 장점과 헥사고날 아키텍쳐의 철학을 잘 이행하고 있는지 궁금해졌습니다. 

이번 포스팅에서는 각 클래스가 지녀야되는 책임, 그리고 잘못된 부분이 있으면 수정을 해보면서 의존성 분리에 한층 더 다가가보도록 하겠습니다. 

## 2. Application Service 와 Handler 
Application Service는 비즈니스 로직을 오케스트레이션 하는 중심 계층입니다. 
	오케스트레이션이란 서비스를 조율하고 관리하는 것으로 이해하면 좋습니다. 
즉, 서비스가 제공해야 하는 유스 케이스 (사용자 생성, 업데이트) 등을 정의하고 이를 실행하기 위해 필요한 작업들을 조합하고 트랙섹션을 관리하는 역할을 합니다. 

예제 코드를 통해 설명해보겠습니다. 
```java
@Transactional
public CreateUserResponse createUser(CreateUserCommand createUserCommand) {
    try {
        // 1. 사용자 정보 저장
        User user = userApplicationHandler.createUser(createUserCommand);
        
        // 2. 초기 프로필 설정 저장
        userApplicationHandler.saveInitialProfileSettings(user);

        // 3. 기본 권한 할당
        userApplicationHandler.assignDefaultRoles(user);

        // 4. 이메일 인증 토큰 생성 및 전송
        userApplicationHandler.sendEmailVerification(user);

        // 5. 이벤트 발행
        userApplicationHandler.publishUserCreatedEvent(user);

        return userDataMapper.toCreateResponse(user, "사용자가 성공적으로 생성되었습니다.");
    } catch (Exception e) {
        log.error("사용자 생성 중 오류 발생: ", e);
        throw new UserCreationException("사용자 생성 중 오류가 발생했습니다.", e);
    }
}
```

참고로 `@Transactional`은 각 메소드에 붙여도 되고 `ApplicationService`의 메소드에 붙여도 되지만 유스 케이스 전체 흐름을 하나의 트랙섹션으로 묶기 위해 `ApplicaionService`에 붙이는게 좋습니다. 

위와 같이 코드를 짰을 때 장점 첫 번째는 책임 분리가 명확하다는 것입니다. 
`ApplicaionService`가 전체적인 흐름을 관리했다면 `ApplicationHandler`는 **구체적인 로직 실행**과 **비즈니스 규칙 적용**을 담당하면 됩니다.

이렇게 책임을 나눔으로써 코드의 응집도가 높아지고, 변경사항이 있을 때 해당 클래스만 변경하면 됩니다. 예를 들어, 기능을 추가할 때는 `ApplicationService`에 추가하면 될 것이고, 새로운 사용자 등록 로직이 추가된다면 `ApplicationHandler`에 등록하면 됩니다. 

유연하게 구현체를 바꿀 수도 있습니다. 물론 이 작업은 UserCreatedHandler의 추상화를 필요로 합니다. 
```java
@Configuration
public class UserServiceConfig {

    @Bean
    public UserCreateHandler userCreateHandler() {
        if (someCondition) {
            return new AdvancedUserCreateHandler(); // 새로운 핸들러 구현체
        } else {
            return new BasicUserCreateHandler(); // 기존 핸들러 구현체
        }
    }
}

```

또한, 재사용성이 높습니다. 이벤트 발행 메소드가 `ApplicationHandler`에 정의되어 있다면, 생성, 수정, 삭제 등의 여러 유스케이스에 해당 메소드를 사용하면 되는 것이죠. 

테스트도 용이합니다. 두 계층을 분리함으로써 각 계층을 독립적으로 테스트 할 수 있기 때문입니다. 

예컨대 `ApplicaionService`는  비즈니스 로직의 테스트 보다는 서비스의 흐름과 상호작용을 Mock을 통해 테스트 합니다.
```java
@SpringBootTest
public class UserApplicationServiceTest {

    @MockBean
    private UserCreateHandler userCreateHandler; // 핸들러를 Mock으로 주입
    @MockBean
    private UserUpdateHandler userUpdateHandler; // 핸들러를 Mock으로 주입

    @Autowired
    private UserApplicationService userApplicationService; // 실제 서비스 사용

    @Test
    public void testCreateUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest("john.doe@example.com", "John Doe");
        User mockUser = new User("john.doe@example.com", "John Doe");
        Mockito.when(userCreateHandler.createUser(request)).thenReturn(mockUser);

        // When
        CreateUserResponse response = userApplicationService.createUser(request);

        // Then
        assertEquals("John Doe", response.getName());
        assertEquals("john.doe@example.com", response.getEmail());
    }
}
```

반면 `ApplicationHandler`는 비즈니스 로직을 테스트하죠.
```java
@SpringBootTest
public class UserCreateHandlerTest {

    @MockBean
    private UserRepository userRepository; // 리포지토리를 Mock으로 주입
    @MockBean
    private UserProfileRepository userProfileRepository; // 리포지토리를 Mock으로 주입

    @Autowired
    private UserCreateHandler userCreateHandler; // 실제 핸들러 사용

    @Test
    public void testCreateUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest("john.doe@example.com", "John Doe");
        User mockUser = new User("john.doe@example.com", "John Doe");

        // 리포지토리 동작 정의
        Mockito.when(userRepository.save(Mockito.any(User.class))).thenReturn(mockUser);

        // When
        User createdUser = userCreateHandler.createUser(request);

        // Then
        assertEquals("John Doe", createdUser.getName());
        assertEquals("john.doe@example.com", createdUser.getEmail());
    }
}
```

개인적으로 `ApplicationHandler`에서 Mapper를 통해 DTO 매핑 하는 것을 선호합니다.
그렇게 된다면 `UserApplicationService`는 유스 케이스의 흐름을 관리하는데 조금 더 집중 할 수 있죠.
```java
@Slf4j
@RequiredArgsConstructor
@ApplicationService
public class UserApplicationService implements UserApplicationUseCase {

    private final UserCreateHandler userCreateHandler;
    private final UserUpdateHandler userUpdateHandler;

    @Override
    public CreateUserResponse createUser(CreateUserRequest createUserRequest) {
        return userCreateHandler.createUser(createUserRequest);
    }

    @Override
    public UpdateUserResponse updateUser(UpdateUserRequest updateUserRequest) {
        return userUpdateHandler.updateUser(updateUserRequest);
    }

    // 기타 유스 케이스 메서드...
}
```

## 3. Handler와 Domain Service 
어쩌면 `ApplicaionService`와 `Handler` 보다 조금 더 분별하기 쉽습니다.
`DomainService`는 어떤 외부와의 연결도 없습니다. 오로지 도메인 로직에 집중합니다. 

예를 들어, 유저를 업데이트 한다고 가정해보겠습니다. 
```java
public class UserApplicationHandler {
	private final UserRepository userRepository;
	private final UserDataMapper userDataMapper;
	private final UserDomainService userDomainService; 
	private final UserEventPublisher userEventPublisher;

	@Transactional
	public UpdateUserResponse updateUser(UpdateUserCommand updateUserCommand) {
		User user = userRepository.findById(updateUserCommand.getUserId())
		.orElseThrow(() -> new UserNotFoundException());

		userDomainService.updateUser(user, updateUserRequest);

		userRepository.save(user);

		return userDataMapper.toUpdateResponse(user, "사용자의 정보가 성공적으로 업데이트 되었습니다.");
	}
}

public class UserDomainService {
	public void updateUser(User user, UpdateUserCommand updateUserCommand) { 
		user.updateName(updateUserCommand.getName());
		user.updateEmail(updateUserCommand.getEmail()); 
}
```