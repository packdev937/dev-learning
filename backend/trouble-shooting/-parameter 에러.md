 ```
Name for argument of type [java.lang.String] not specified, and parameter name information not available via reflection. Ensure that the compiler uses the '-parameters' flag.]
```

위 에러는 자바 컴파일러가 메서드의 파라미터 이름을 바이트 코드에 포함하지 않았기 때문에 발생합니다. 

예를 들어 설명해보겠습니다. 
```java
public void sayHello(String name) {
	Syetem.out.println("Hi! My name is ", name);
}
```

위 메서드를 컴파일하면, 자바는 기본적으로 파라미터의 이름을 유지하지 않 `arg0`, `arg1`, ... 과 같은 이름으로 변경합니다. 즉, 파라미터 타입 정보만 남고 이름은 유지되지 않습니다. 

```java
public void sayHello(String arg0) {
	Syetem.out.println("Hi! My name is ", arg0);
}
```

이렇게 되면 리플렉션을 통해 메서드를 호출할 때 파라미터의 이름이 아닌 타입 정보만 알 수 있습니다. 

`-parameters` 플래그를 사용하면, **컴파일된 바이트코드에 메서드의 파라미터 이름을 그대로 포함**할 수 있습니다. 즉, 컴파일된 후에도 `userId`와 `age`라는 이름이 유지되며, 이를 리플렉션이나 프레임워크(Sprint 등)를 통해 참조할 수 있게 됩니다.

`-parameters` 플래그는 `build.gradle`에 다음과 같이 추가 가능합니다. 
```
tasks.withType(JavaCompile) { 
	options.compilerArgs << '-parameters' 
}
```

### 리플렉션을 강조하는 이유는?

Spring과 같은 프레임워크는 **리플렉션**을 통해 컨트롤러 메서드에 있는 파라미터 정보를 **런타임에 해석**하고, HTTP 요청 파라미터를 메서드 파라미터에 바인딩합니다. 다음 예시처럼 Spring에서 리플렉션을 통해 메서드의 파라미터 이름을 필요로 하는 상황이 있습니다.

```java
@DeleteMapping("/users")
public ResponseEntity<Void> deleteUser(
	@RequestParam String userId
) {

}
```

이 코드에서 Spring은 **리플렉션을 사용해 `deleteUser` 메서드의 파라미터 이름(`userId`)을 알아내고, HTTP 요청에서 해당 파라미터 값을 추출**하여 메서드에 전달하려고 합니다.

그러나 컴파일 시에 **파라미터 이름이 바이트코드에 포함되지 않으면**, Spring은 리플렉션을 통해 파라미터 이름을 확인할 수 없고, 대신 `arg0`과 같은 기본 이름을 사용하려고 시도합니다. 이때 파라미터 이름 정보를 얻지 못해 예외가 발생할 수 있습니다.
### 어떤 경우에 다음과 같은 에러가 발생했는가?

### 1 
@RequestParam 시 매핑 할 쿼리를 제대로 명시하지 않았을 때 예외가 발생했습니다. 
```java
@GetMapping("/validation/id")  
ResponseEntity<ValidateUserResponse> validateUserId(  
    @RequestParam(name = "id") String id);
```

따라서 build.gradle에 속성을 추가하거나  위와 같이 name (또는 value) 속성을 지정해주면 됩니다. 

### 2
커스텀 어노테이션 사용 시 오류가 계속해서 발생했습니다.
```java
@Operation(summary = "유저 삭제", description = "유저를 삭제합니다.")  
@DeleteMapping  
Mono<ResponseEntity<DeleteUserResponse>> deleteUser(  
    @CurrentUserId String requestUserId);
```

알고보니 `WebMvc`와 `WebFlux`를 동시에 명시해놓고, 어노테이션의 처리는 `WebFluxConfig`에서 진행하여 문제가 발생했습니다. (중복일 경우 `WebMvc`가 default로 설정) 

이를 해결하기 위해서는 `WebMvc`에 커스텀 어노테이션을 처리할 수 있는 resolver를 등록하거나, `WebMvc` 의존성을 제거하는 방법이 있습니다. 