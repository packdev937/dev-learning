## 1. 개요
서버로 API 요청을 할 때는 대게 JSON 형태로 데이터를 건내줍니다. 이를 서버에서는 `Enum` 혹은 `LocalDate` 형태로 받아야 할 때가 있는데 이 때 `@RequestBody`는 어떻게 작동할까요? 

## 2. 사용 예제 
다음과 Controller 메소드가 있다고 가정해보겠습니다.
```java
@PostMapping("/users")
public CreateUserResponse createUser(
	@Valid @RequestBody CreateUserRequest createUserRequest
);
```

그리고 `CreateUserRequest`는 다음과 같이 작성되어 있습니다.
```java
public record CreateUserRequest(
	String username,
	Role role, 
	LocalDate birthday
)

public Enum Role {
	ROLE_USER("USER"),
	ROLE_ADMIN("ADMIN");


	private String description;
}
```

먼저 **Enum**은 요청의 경우 **문자열**로 전달되면, 그 값이 Enum 타입에 정의된 값과 일치하는 경우 자동으로 매핑됩니다.

위의 경우에서는 USER 또는 ADMIN으로 대소문자 정확히 전달되어야 합니다. 

**날짜**의 경우 ISO 8601 형식으로 문자열로 전달되면 **`LocalDate`, `LocalDateTime`** 등으로 자동 매핑됩니다. 

매핑 방식을 커스텀하기 위해서는 `@JsonFormat`을 사용할 수 있습니다. 

## 3. 동작 과정
스프링에서 요청 데이터를 자동으로 특정 타입으로 변환할 수 있는 이유는 Message Converter라는 내부 동작 메커니즘 덕분입니다. 

### 3-1. HttpMessageConverter
스프링은 클라이언트로부터 요청 본문 데이터를 받아 DTO나 자바 객체로 변환할 때 **`HttpMessageConverter`** 인터페이스를 사용합니다. 이 인터페이스는 다양한 형식의 요청 본문을 처리할 수 있도록 여러 구현체를 제공합니다.

- **`MappingJackson2HttpMessageConverter`**: 주로 JSON 형식의 데이터를 자바 객체로 변환합니다. 
- **`StringHttpMessageConverter`**: 요청 본문이 문자열로 된 경우 이를 자바 문자열로 변환합니다.
- **`FormHttpMessageConverter`**: `application/x-www-form-urlencoded`나 `multipart/form-data` 형식의 요청을 처리합니다.

### 3-2. JSON 파싱과 Enum 변환
JSON 데이터를 자바 객체로 변환하는 기본적인 메커니즘은 Jackson 라이브러리입니다. 스프링 부트는 `MappingJackson2HttpMessageConverter`를 기본적으로 사용해 JSON 형식의 데이터를 자바 객체로 변환합니다.

Enum은 JSON에서 전달된 문자열 값이 자바 객체의 Enum 필드에 매핑될 때, Jackson은 그 문자열이 Enum에서 정의된 값과 일치하는지 확인하고, 일치할 경우 자동으로 해당 Enum 타입으로 변환합니다.

예를 들어 `{"status": "PENDING"}`이라는 JSON 요청이 들어오면, `OrderStatus.PENDING` Enum 값으로 매핑됩니다.
    
내부적으로 Jackson은 `Enum.valueOf()` 메서드를 사용하여 문자열을 Enum 타입으로 변환합니다.

### 3-3. **날짜 변환**
날짜 변환은 Jackson이 기본적으로 ISO 8601 형식(예: `"yyyy-MM-dd"` 또는 `"yyyy-MM-dd'T'HH:mm:ss"`)을 사용하여 JSON의 날짜 형식 데이터를 자바의 `LocalDate`나 `LocalDateTime`으로 변환합니다.

예를 들어, JSON 요청에서 `"eventDate": "2024-09-08"`와 같이 들어오면, Jackson은 이를 `LocalDate.of(2024, 9, 8)`로 자동 변환합니다.
