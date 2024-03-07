간단하게 Swagger를 설정하는 법을 알아보겠습니다.

`build.gradle`에 의존성 추가
```
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0'
```

`SwaggerConfig.java` 생성
```java
@OpenAPIDefinition(  
    info = @Info(  
        title = "GDSC Solution Challenge : Sunshine",  
        description = "Sunshine API 명세서",  
        version = "v1"  
    )  
)  
@Configuration  
public class SwaggerConfig {  
  
    @Bean  
    public OpenAPI openAPI() {  
        return new OpenAPI()  
	            .servers(List.of(new Server().url("http://localhost:8080/").description("Sunshine API Server")));  
    }  
}
```


`application.yml`에 설정 추가
```
springdoc:  
  swagger-ui:  
    path: /swagger-ui.html  
    groups-order: DESC  
    operationsSorter: method  
    disable-swagger-default-url: true  
    display-request-duration: true  
  api-docs:  
    path: /api-docs  
  show-actuator: true  
  default-consumes-media-type: application/json  
  default-produces-media-type: application/json  
  paths-to-match:  
    - /v1/**
```

Controller
```java
@Tag(name = "User", description = "User API")  
public class UserController {
```

만약 Token 기반 인증을 추가하고 싶다면?
```
@SecuritySchemes({  
        @SecurityScheme(  
                name = SECURITY_SCHEME_BEARER,  
                type = SecuritySchemeType.HTTP,  
                bearerFormat = "JWT",  
                scheme = "bearer",  
                description = "인증이 필요한 경우 사용될 token 을 넣어준다."  
  
        )  
})
```
