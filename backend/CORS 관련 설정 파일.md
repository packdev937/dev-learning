WebConfig.java
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

그래도 안된다면 
``` java
@CrossOrigin(origins = "https://kobaco.vercel.app", allowedHeaders = "*")
```
