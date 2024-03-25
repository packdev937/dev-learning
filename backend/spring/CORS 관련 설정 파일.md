`WebConfig.java`
```java
@Configuration  
public class WebConfig implements WebMvcConfigurer {  
  
    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
        registry.addMapping("/**")  
            .allowedOrigins("http://localhost:3000", "https://kobaco.vercel.app")  
            .allowedHeaders("*")  
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")  
            .allowCredentials(true)  
            .maxAge(3600);  
    }  
}
```

풀어서 해석하면 다음과 같습니다. 
- `addMapping("/**")`: 애플리케이션의 모든 경로에 대해 CORS 정책을 적용합니다.
- `allowedOrigins(...)`: 허용되는 출처를 지정합니다. 여기서는 "http://localhost:3000"과 "https://kobaco.vercel.app"이 허용됩니다.
- `allowedHeaders("*")`: 모든 헤더를 허용합니다.
- `allowedMethods(...)`: 허용되는 HTTP 메소드를 지정합니다.
- `allowCredentials(true)`: 크레덴셜을 허용합니다.
- `maxAge(3600)`: 사전 요청의 결과를 3600초 동안 캐시합니다.

그래도 안된다면 다음을 붙여주자 
``` java
@CrossOrigin(origins = "https://kobaco.vercel.app", allowedHeaders = "*")
```
