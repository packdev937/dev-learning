> REST 기반 웹 서비스를 테스트 하기 위한 Java 라이브러리 입니다. 

외부 사용자의 관점에서 REST API 자동화 테스트를 구상하는 것은 시스템의 안정성에 큰 도움을 줄 수 있습니다. 하지만 프로젝트의 규모에 따라 고민의 여지는 분명히 존재합니다.

사용하기에 앞서 의존성을 추가해 줍니다. 
```
testImplementation 'io.rest-assured:rest-assured'
```

**에제 코드** 
```java
private static ExtractableResponse<Response> 상품등록요청(  
    AddProductRequest request) {  
    final ExtractableResponse<Response> response = RestAssured
		.given().log().all()  
        .contentType(MediaType.APPLICATION_JSON_VALUE)  
        .body(request)  
        .when()  
        .post("/products")  
        .then().log().all()
        .extract();  
    return response;  
}
```

given() 을 호출하면 RequestSpecification 객체가 생성됩니다. 이를 통해 다양한 요청 값을 설정할 수 있습니다.
	더 나아가 RequestSpecification에 대해 공부하고 싶다면 다음을 참고하면 좋을 것 같습니다.
	https://velog.io/@dahunyoo/SpecBuilder-in-REST-Assured

when()은 HTTP Resource를 지정합니다. HTTP Method와 경로를 함께 지정합니다.

extract()를 통해 ExtractableResponse를 반환 할 수 있습니다. 

이 때, ResponseBodyExtractionOptions의 JsonPath()를 사용하여 JsonPath를 얻을 수 있습니다. 
JsonPath의 getString(), getList()와 같은 메서드를 통해 값이 제데로 들어갔는지 테스트 할 수 있습니다. 

따라서 JsonPath와 RestAssured를 함께 사용하여 JSON 파일의 데이터를 추출 할 수 있습니다.

![[ExtractableResponse.png]]


### Reference
- https://kookiencream.tistory.com/115
- https://loopstudy.tistory.com/427