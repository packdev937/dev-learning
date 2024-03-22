**ErrorCode 정의**
```java
@Getter
@AllArgsConstructor
public enum ErrorCode {

    USER_ALREADY_EXIST(409, "1001", "유저가 이미 존재합니다."),
    USER_NOT_FOUND(404, "1002", "유저를 찾을수 없습니다."),
    INVALID_PASSWORD(401, "1003", "패스워드가 틀렸습니다."),
    DUPLICATED_USER_NAME(409, "1004", "중복된 아이디입니다."),
    INVALID_TOKEN(401, "1005", "토큰이 유효하지 않습니다."),
    INVALID_PERMISSION(401, "1006", "접근권한이 없습니다."),
    INTERNAL_SERVER_ERROR(500, "1008", "서버에 에러가 발생했습니다.")
    ;

    private int status;
    private String code;
    private String message;

}
```

커스텀 예외 클래스 정의 
``` java
@Getter @AllArgsConstructor 
public class UserApplicationException extends RuntimeException { 
	
	private ErrorCode errorCode; 
	
}
```

GlobalControllerAdvice 생성 
```java
@Slf4j
@RestControllerAdvice 
public class GlobalControllerAdvice { /** * 에러 중앙처리 로직 작성완료 */ 

	@ExceptionHandler(UserApplicationException.class) 
	public ResponseEntity<?> applicationHandler(UserApplicationException e) { 
	
		log.error("Error occurs {}", e.toString()); // 에러 응답을 세팅한다. 
		Map<String, String> errorResponse = new HashMap<>(); 
		errorResponse.put("code", e.getErrorCode().getCode()); 
		errorResponse.put("message", e.getErrorCode().getMessage()); return 
		ResponseEntity.status(e.getErrorCode().getStatus()) .body(errorResponse); 
		}
	
```