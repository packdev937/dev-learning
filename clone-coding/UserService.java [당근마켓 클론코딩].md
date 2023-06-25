# UserService.java [당근마켓 클론코딩]

### 코드 
```java
@Service
@Validated
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final JwtUtil jwtUtil;
    private final PasswordEncoder passwordEncoder;

    @Transactional
    public String signup(@Valid SignupRequestDto signupRequestDto){
        String username = signupRequestDto.getUsername();
        String password = passwordEncoder.encode(signupRequestDto.getPassword());
        String nickName = signupRequestDto.getNickName();

        Optional<User> foundUsername = userRepository.findByUsername(username);
        if (foundUsername.isPresent()) {
            throw new IllegalArgumentException("이미 가입된 사용지입니다.");
        }
        Optional<User> foundNickname = userRepository.findByNickName(nickName);
        if (foundNickname.isPresent()) {
            throw new IllegalArgumentException("이미 존재하는 닉네임입니다.");
        }

        User user = new User(username, password, nickName);
        userRepository.save(user);
        return "회원가입 완료";
    }

    public String login(LoginRequestDto loginRequestDto, HttpServletResponse response) {
        String username = loginRequestDto.getUsername();
        String password = loginRequestDto.getPassword();

        User user = userRepository.findByUsername(username).orElseThrow(
                () -> new IllegalArgumentException("등록된 사용자가 없습니다.")
        );

        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new IllegalArgumentException("비밀 번호가 옳지 않습니다.");
        }

        response.addHeader(JwtUtil.AUTHORIZATION_HEADER, jwtUtil.createToken(user.getNickName()));
        return "로그인 성공";
    }

    public UserResponseDto getUser(UserDetailsImpl userDetails) {
        User user = userRepository.findByUsername(userDetails.getUsername()).orElseThrow(
                () -> new IllegalArgumentException("등록된 사용자가 없습니다.")
        );

        return new UserResponseDto(user);
    }
}
```

### ✌️우선 **회원가입 로직**을 확인해보자

```java
@Transactional
    public String signup(@Valid SignupRequestDto signupRequestDto){
        String username = signupRequestDto.getUsername();
        String password = passwordEncoder.encode(signupRequestDto.getPassword());
        String nickName = signupRequestDto.getNickName();

        Optional<User> foundUsername = userRepository.findByUsername(username);
        if (foundUsername.isPresent()) {
            throw new IllegalArgumentException("이미 가입된 사용지입니다.");
        }
        Optional<User> foundNickname = userRepository.findByNickName(nickName);
        if (foundNickname.isPresent()) {
            throw new IllegalArgumentException("이미 존재하는 닉네임입니다.");
        }

        User user = new User(username, password, nickName);
        userRepository.save(user);
        return "회원가입 완료";
    }
```

흐름은 다음과 같다.

DTO를 이용해 검증을 거친다. → DTO로 넘어온 데이터들을 새로운 문자열에 담는다

→ 이 때 비밀번호는 passwordEncoder를 통해 인코딩 한다

→ 아이디와 닉네임은 겹치면 안되므로 저장소에 커스텀한 메소드로 중복 여부를 확인한다

→ 무사히 통과했다면 repository에 저장한다.

### @Valid는 Controller 단에서 처리하는게 좋은가 Service 단에서 처리하는게 좋은가?

조사에 따르면 각각의 layer에서 적절한 유효성 검사를 하는게 타당합니다.

**참조 문헌**

[[Spring] Validation은 어디서 해야할까?](https://velog.io/@rmswjdtn/Spring-Validation은-어디서-해야할까)

### PasswordEncoder

```
public interface PasswordEncoder {

　　// 비밀번호를 단방향 암호화
　　String encode(CharSequence rawPassword);

　　// 암호화되지 않은 비밀번호(raw-)와 암호화된 비밀번호(encoded-)가 일치하는지 비교
　　boolean matches(CharSequence rawPassword, String encodedPassword);

　　// 암호화된 비밀번호를 다시 암호화하고자 할 경우 true를 return하게 설정
　　default boolean upgradeEncoding(String encodedPassword) { return false; };
}
```

다음과 같은 메소드들이 구현되어 있다 정도만 알고 넘어갑시다.

위 클론 코딩 코드를 보면 회원 가입 때는 encode()를 이용하여 비밀번호를 암호화 한 뒤 repository에 저장하고, 로그인 때는 encoding된 password를 matches()를 사용해 로그인 입력 시 password와 비교하여 동일 여부를 확인합니다.

**참조 문헌**

[[Spring Security] PasswordEncoder란?](https://velog.io/@corgi/Spring-Security-PasswordEncoder란-4kkyw8gi)

### Optional 제대로 사용하기

먼저 Optional이 등장한 이유부터 알아봅시다.

> ***Optional is primarily intended for use as a method return type where there is a clear need to represent "no result," and where using null is likely to cause errors. A variable whose type is Optional should never itself be null; it should always point to an Optional instance.***
>

쉽게 말해서 null로 인한 `**NullPointerException`** 을 슬기롭게 대처하고자 Optional이 탄생했습니다.

**Optional**을 올바르게 사용하는 방법

1. 절대 null을 할당하지 말 것

```java
Optional<Member> findById(Long id) {
    if (result == 0) {
        //return null;
				return Optional.empty();
    }
}
```

결과 없음(no result)을 표현하고 쉽다면 null 대신 Optional.empty()를 사용합시다.

1. Optional 객체의 값 보유 여부를 확인할 것

빈 Optional 객체에 get()을 한다면 NoSuchElementException이 발생합니다. 따라서 값을 가져오기 이전에 값의 유무를 반드시 확인해야 합니다.

```java
Member newMember = userRepository.findById(1L).orElseThrow(
			() -> new IllegalArgumentException("등록된 사용자가 없습니다.")
);
```

더 자세한 내용은 다음을 확인하자

**참조 문헌**

[[Java] Optional 올바르게 사용하기](https://dev-coco.tistory.com/178)

### 로그인 로직을 확인해보자

```java
public String login(LoginRequestDto loginRequestDto, HttpServletResponse response) {
        String username = loginRequestDto.getUsername();
        String password = loginRequestDto.getPassword();

        User user = userRepository.findByUsername(username).orElseThrow(
                () -> new IllegalArgumentException("등록된 사용자가 없습니다.")
        );

        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new IllegalArgumentException("비밀 번호가 옳지 않습니다.");
        }

        response.addHeader(JwtUtil.AUTHORIZATION_HEADER, jwtUtil.createToken(user.getNickName()));
        return "로그인 성공";
    }
```

흐름은 다음과 같다.

DTO로 넘어온 username과 password를 사용해서 사용자 검증을 한다 (회원 가입이 되었는지, 비밀번호가 틀리지 않았는지)

→ 만약 검증이 되었다면 JWT 기반의 인증 토큰을 생성하여 HTTP 응답 헤더에 추가한다.

### 언급된 김에 알아보는 HttpServletResponse

우선 addHeader() 안에 두 매개변수가 뭘 의미하는지 알아보자.

`JwtUtil.AUTHORIZATION_HEADER`는 “Authorization” 을 의미한다.

`JwtUtil.java`

```java
public static final String AUTHORIZATION_HEADER = "Authorization";
```

`jwtUtil.createToken()`은 닉네임을 기반으로 한 Jwt Token을 만드는 메소드이다.

따라서 위 함수를 실행하면 응답 헤더에 Authorization : [JWT Token] 이 추가될 것이다.

**참조 문헌**

[Servlet 응답 정보 처리 (HttpServletResponse, 한글 응답)](https://kgvovc.tistory.com/30)

### DTO는 어떻게 다른가?

위 코드를 보면 회원 가입 시 DTO와 로그인 시 DTO가 다른 것을 확인할 수 있다. 어떻게 다르게 구성되어 있을까?

```java
@Getter
public class LoginRequestDto {

    private String username;
    private String password;
}

@Getter
public class SignupRequestDto {

    @Size(min = 4, max = 10)
    private String username;

    @Pattern(regexp = "(?=.*?[a-zA-Z])(?=.*?[\\d])(?=.*?[~!@#$%^&*()_+=\\-`]).{8,15}")
    private String password;

    private String nickName;
}
```

회원 가입 시에는 @Validation 어노테이션이 붙어 있으며, username, password, nickname을 입력받지만, 로그인 시에는 username과 password만 요청한다.

마지막 user의 정보를 불러오는 getUser 메소드도 위와 비슷하게 필요한 정보만 담아서 반환하고 있다.

```java
public UserResponseDto getUser(UserDetailsImpl userDetails) {
        User user = userRepository.findByUsername(userDetails.getUsername()).orElseThrow(
                () -> new IllegalArgumentException("등록된 사용자가 없습니다.")
        );

        return new UserResponseDto(user);
    }

@Getter
public class UserResponseDto {
    private String username;
    private String nickName;

    public UserResponseDto(User user) {
        this.username = user.getUsername();
        this.nickName = user.getNickName();
    }

    public String getUsername() {
        return username;
    }

    public String getNickName(){
        return nickName;
    }
}
```