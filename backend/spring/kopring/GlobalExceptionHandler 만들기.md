```kotlin
data class ErrorResponse (
	val message: String, 
	val errorType: String = "Invalid Argument"
)
```

참고로 errorType에 있는 값은 Default 값이며 별도의 선언이 없는 경우 해당 값이 사용됩니다. 
두 변수는 모두 val로 선언되었으니, 초기화 이후에는 재 할당이 불가능합니다.

```kotlin
class InvalidInputException(
	message: String = "Invalid Input"
): RuntimeException(message)
```

위 코드를 자바로 나타내면 다음과 같습니다.
```java
public class InvalidInputException extends RuntimeException { 

	public InvalidInputException() { 
		super("Invalid Input"); 
	} 
		
	public InvalidInputException(String message) { 
		super(message); 
	} 
}
```

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler {

	@ExceptionHandler(InvalidInputException::class)
	fun invalidInputException(ex: InvalidInputException): ResponseEntity<ErrorResponse> {
		val errors = ErrorResponse(ex.message ?: "Not Exception Message")
		return ResponseEntity(errors, HttpStatus.BAD_REQUEST)
	}
}
```
