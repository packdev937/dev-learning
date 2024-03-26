Swagger 라이브러리는 두 가지 종류가 있으며 종류는 다음과 같습니다. 
- Spring-fox 
- Spring-doc 

Spring-fox는 2020년 이후 별다른 업데이트가 없다고 하니 이번 실습에서는 Spring-doc을 사용해보겠습니다. 

#### build.gradle
Swagger를 적용하기 위해서는 먼저 `build.gradle`에 **dependency를** 추가해주어야 합니다. 

``` 
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui'
```

#### application.yml 
그 다음으로는 yml 속성 파일을 작성해줍니다. 대게 많이 사용하는 속성은 다음과 같습니다. 
``` yaml
springdoc:  
  swagger-ui:
	# Swagger UI 페이지에 대한 경로 
    path: /swagger-ui.html  
    # Group 별 정렬 기준 (ASC 오름차순, DESC 내림차순)
    groups-order: DESC  
    # API 목록 내 정렬 기준 (method, alpha)
    operationsSorter: method  
    # Swagger UI가 로드될 때 기본적으로 제공하는 예제 URL 생략
    disable-swagger-default-url: true  
    # API 요청 시 실행 시간을 표시
    display-request-duration: true  
  api-docs:  
    path: /api-docs  
  show-actuator: true  
  default-consumes-media-type: application/json  
  default-produces-media-type: application/json  
  paths-to-match:  
    - /v1/**
```

#### SwaggerConfig.java
의존성을 추가한 다음 해야할 일은 Swagger 설정 파일을 생성하는 것입니다. 
```java
@OpenAPIDefinition(  
    info = @Info(  
        title = "Swagger API",  
        version = "1.0",  
        description = "Documentation Swagger API v1.0",  
        license = @License(name = "Apache 2.0", url = "http://springdoc.org"),  
        contact = @Contact(name = "Swagger Support", email = "packdev937@gmail.com")  
    ),  
    servers = {  
        @Server(url = "http://localhost:8080/", description = "Local 8080 Server"),  
        @Server(url = "https://localhost:8081/", description = "Local 8081 Server")  
    }  
)  
@SecuritySchemes({  
    @SecurityScheme(  
        name = "Bearer Token",  
        type = SecuritySchemeType.HTTP,  
        bearerFormat = "JWT",  
        scheme = "bearer",  
        description = "JWT Bearer Token.."  
    )  
})  
@Configuration  
public class SwaggerConfig {  
  
    // 그룹핑  
    @Bean  
    public GroupedOpenApi swaggerOpenApi() {  
        String[] paths = {"/v1/swagger/**"};  
  
        return GroupedOpenApi.builder()  
            .group("Swagger 서비스 API v1")  
            .pathsToMatch(paths)  
            .build();  
    }  
  
    @Bean  
    public GroupedOpenApi practiceOpenApi() {  
        String[] paths = {"/v1/practice/**", "/v1/best/**"};  
  
        return GroupedOpenApi.builder()  
            .group("Practice 서비스 API v1")  
            .pathsToMatch(paths)  
            .build();  
    }  
}
```

적용할 수 있는 모든 설정을 적용해보았습니다. 

#### Controller.java
```java
@RestController  
@Tag(name = "Swagger API", description = "Documentation Swagger API v1.0")  
public class SwaggerController {  
  
    @Operation(summary = "회원 탈퇴 요청", description = "회원 정보가 삭제됩니다.", tags = { "Member Controller" })  
    @ApiResponses({  
        @ApiResponse(responseCode = "200", description = "OK",  
            content = @Content(schema = @Schema(implementation = String.class))),  
        @ApiResponse(responseCode = "400", description = "BAD REQUEST"),  
        @ApiResponse(responseCode = "404", description = "NOT FOUND"),  
        @ApiResponse(responseCode = "500", description = "INTERNAL SERVER ERROR")  
    })    @GetMapping("/swagger")  
    public String swagger() {  
        return "Swagger API";  
    }  
}
```