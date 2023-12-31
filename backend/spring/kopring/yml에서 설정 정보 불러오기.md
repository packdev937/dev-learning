`application.yml`에 다음 정보가 있다고 가정해봅시다.
```kotlin
...
REST_API_KEY = "dsf934ifdj0.."
```

이를 다음과 같이 가져올 수 있습니다.
```kotlin
@Service
class BlogService {

	@Value("\${REST_API_KEY}")
	lateinit var restApiKey: String
	...
}
```

\ 가 포함된 이유는 다음과 같습니다.
> $가 문자열 템플릿을 나타내는 것이 아닌 문자열의 일부로 취급되어야 함 

lateinit은 실행 시점에 변수가 초기화 될 것을 알려줍니다.

따라서 위 코드는 설정 파일에서 `REST_API_KEY` 값을 읽어와서 `restApiKey` 변수에 나중에 할당합니다.