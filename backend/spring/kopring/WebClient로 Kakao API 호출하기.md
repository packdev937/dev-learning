우선 WebClient에 대한 간단한 이해가 필요합니다. 
이는 다음을 참고하면 됩니다. 
[[WebClient vs RestTemplate]]

WebClinet를 사용하기 위해서는 SpringWebFlux 의존성을 추가해야 합니다.
```kotlin
implementation 'org.springframework:spring-webflux'
```

**예제 코드**
```kotlin
var webClient = WebClient
		.builder()
		.baseUrl("https://dapi.kakao.com")
		.defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
		.build()

var response = webClient()
		.get()
		.uri { it.path("/v2/search/blog")
			.queryParam("query", blogDto.query)
			.queryParam("sort", blogDto.query)
			.queryparam("page", blogDto.page)
			.queryParam("size", blogDto.size)
			.build()}
		.header("Authorization", REST_API_KEY)
		.retrieve()
		.bodyToMono<String>()

var result = response.block()

return result
```

각각에 대한 설명은 다음과 같습니다.
**WebClient.builder()**
    이 메소드는 `WebClient`의 빌더를 생성합니다. 빌더 패턴은 객체 생성을 단계별로 수행하며, 최종적으로 완성된 객체를 제공합니다.

**baseUrl(String baseUrl)**
	기본 URL을 설정합니다.

**build()**
	설정된 옵션들을 바탕으로 `WebClient` 인스턴스를 생성합니다.

**get()**
	HTTP GET 요청을 수행합니다.

**uri(Function<UriBuilder, URI> uriFunction)**:
	 요청할 URI를 설정합니다. `UriBuilder`를 사용하여 경로(`/v2/search/blog`)와 쿼리 파라미터들(`query`, `sort`, `page`, `size`)을 설정합니다.

**header(String headerName, String headerValue)**:
	특정 요청에 헤더를 추가합니다. 이 경우 `Authorization` 헤더가 추가되며, 그 값으로는 `REST_API_KEY`가 사용됩니다.

 **retrieve()**
	 요청을 보내고 응답을 검색합니다. 이 메소드는 응답을 처리하기 위한 다양한 메소드들을 제공합니다.

bodyToMono(Class<T> elementClass)
	응답 본문을 지정된 클래스 타입의 객체로 변환합니다. 여기서는 `String` 타입으로 변환됩니다

block()
	`Mono`의 결과를 동기적으로 가져옵니다. 이것은 요청이 완료될 때까지 현재 스레드를 블록합니다.

