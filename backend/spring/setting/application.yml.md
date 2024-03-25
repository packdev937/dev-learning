**database**
``` yml
spring:  
  datasource:  
      driver-class-name: com.mysql.cj.jdbc.Driver  
      username: # 데이터베이스 커넥션 이름  
      password: # 데이터베이스 커넥션 비밀번호  
      url: jdbc:mysql:[url 주소]?useUnicode=true&serverTimezone=Asia/Seoul
```

**swagger** 
``` yml
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
    - /api/v1/**
```

secrets.yml
```yml
spring:  
  config:  
    import: optional:secrets.yml
```

dev, local, prod로 나눈다면 (application-dev.yml)
``` yml
spring:
	profiles:  
		active: local
```

전송하는 파일의 크기가 크다면 
``` bash
spring:
	servlet:  
	  multipart:  
	    max-file-size: 10MB  
	    max-request-size: 10MB
```

**Spring Batch** 
```bash
# application.properties
spring.batch.jdbc.initialize-schema=always

# application.yml
spring:
  batch:
    jdbc:
      initialize-schema=always
```
