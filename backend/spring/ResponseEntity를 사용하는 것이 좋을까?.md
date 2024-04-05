REST API를 구축한다면 서버 통신 간 필요한 정보를 제공해야 합니다. 

만약 객체를 반환하는 경우 HTTP 응답을 반환할 수 없습니다. 이 때 ResponseEntity를 사용합니다. 

```java
@GetMapping("")
public ResponseEntity<UserResponse> getUser(@RequestParam(name = "userId") Long userId){
	return ResponseEntity.ok(userService.findById(userId));
}
```

종종 ResponseEntity 대신 Custom 한 반환 객체를 사용하는 경우가 있습니다. 다음이 그 예시입니다. 
```java
@GetMapping(""")  
   public ApiResult<UserDto> login(  
       @AuthenticationPrincipal JwtAuthentication authentication  
   ) {  
       return success(  
           userService.findById(authentication.id)  
			.map(UserDto::new)  
			.orElseThrow(  
			    () -> new NotFoundException("Could nof found user for " + authentication.id))  
       );  
   }
```

내부의 로직은 신경쓰지 말고 흐름을 봅시다. ResponseEntity와 동일하게 success() 안에 실행 결과를 전달하면 HttpStatus.OK를 반환하는 것 같습니다.

```java
public class ApiUtils {  
  
    public static <T> ApiResult<T> success(T response) {  
        return new ApiResult<>(true, response, null);  
    }  
  
    public static ApiResult<?> error(Throwable throwable, HttpStatus status) {  
        return new ApiResult<>(false, null, new ApiError(throwable, status));  
    }  
  
    public static ApiResult<?> error(String message, HttpStatus status) {  
        return new ApiResult<>(false, null, new ApiError(message, status));  
    }  
  
    @Getter  
    @ToString    public static class ApiError {  
        private final String message;  
        private final int status;  
  
        ApiError(Throwable throwable, HttpStatus status) {  
            this(throwable.getMessage(), status);  
        }  
  
        ApiError(String message, HttpStatus status) {  
            this.message = message;  
            this.status = status.value();  
        }  
    }  
  
    @Getter  
    @ToString    public static class ApiResult<T> {  
        private final boolean success;  
        private final T response;  
        private final ApiError error;  
  
        private ApiResult(boolean success, T response, ApiError error) {  
            this.success = success;  
            this.response = response;  
            this.error = error;  
        }  
  
        public boolean isSuccess() {  
            return success;  
        }  
    }  
}
```

그렇다면 어떤 것이 더 좋을까요? 결론은 취향 차이 입니다.