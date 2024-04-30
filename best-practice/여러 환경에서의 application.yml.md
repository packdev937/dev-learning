여러 환경에서 설정 해야 할 때는 다음과 같이 설정 파일을 선택해서 실행할 수 있습니다. 
``` bash 
#spring.profiles.include=local  
#spring.profiles.include=dev  
#spring.profiles.include=prod  
spring:
	profiles:
		include: test
		# include : local
		# include : prod
		# include : dev
```

**서버 포트 설정** 
```yaml
server:
	port: 9000
```

**데이터베이스 설정** (h2)
```yaml
spring:  
  datasource:  
    driver-class-name: org.h2.Driver  
    username: sa  
    password:  
#    url: jdbc:h2:tcp://localhost/~/[Database 명]
     url: jdbc:h2:mem:test;MODE=MySQL # 간편하게 내장 DB를 사용할 수도 있습니다
```


**JPA**
```yaml
spring:
	jpa:  
	  hibernate:  
	    ddl-auto: create-drop  # create, create-drop, update, validate, none
	  properties:  
	    hibernate:  
			format_sql: true # SQL 포맷팅  
			show_sql: true # SQL 출력
```

**파일 크키 설정**
```yaml
spring:
	servlet:
		multipart.max-file-size: 20MB  
		multipart.max-request-size: 20MB  
```

**스웨거**
```yaml
springdoc:  
  swagger-ui:  
    # Swagger UI 페이지에 대한 경로  
    path: /swagger-ui.html  
    # Group 별 정렬 기준 (ASC 오름차순, DESC 내림차순)  
    groups-order: DESC  
    # API 목록 내 정렬 기준 (method, alpha)    operationsSorter: method  
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
    - /api/**
```

많이 사용하는 JWT 토큰 Properties 
```yaml
jwt:  
  secret-key:   
  issuer: 
  access:  
    expiration: 3600000 # 1시간 (1000L(ms -> s) * 60L(s -> m) * 60L(m -> h))
	header: Authorization  
  refresh:  
    expiration: 1209600000 # 2주 (1000L(ms -> s) * 60L(s -> m) * 60L(m -> h) * 24L(h -> 하루) * 14(2주))  
    header: Authorization-refresh
```

**Spring OAuth** 
```yaml
spring:  
  security:  
    oauth2:  
      client:  
        registration:  
          kakao:  
            # 앱키 -> REST API 키  
            client-id: 
            # 카카오 로그인 -> 보안 -> Client Secret 코드  
            client-secret:   
            authorization-grant-type: authorization_code  
            # redirect-uri: https://goodmoneying.shop/login/oauth2/code/kakao  
            # Spring Security에서 기본적으로 제공하는 URL. 별도의 End Point가 필요하지 않습니다.  
            redirect-uri: http://localhost:9000/login/oauth2/code/kakao  
            # POST는 쓸 수 없습니다. POST로 요청을 보내면 405 Method Not Allowed 에러가 발생합니다.  
            client-authentication-method: client_secret_post  
            client-name: kakao  
            # 각 플랫폼의 사용자 동의 체크에서 필요한 정보의 필드를 작성합니다.  
            scope:  
              - profile_nickname  
              - profile_image  
              - account_email  
        provider:  
          kakao:  
            # "인가 코드 받기" 항목  
            authorization-uri: https://kauth.kakao.com/oauth/authorize  
            # "토큰 받기" 항목  
            token-uri: https://kauth.kakao.com/oauth/token  
            # "사용자 정보 가져오기" 항목  
            user-info-uri: https://kapi.kakao.com/v2/user/me  
            # 식별자. 카카오의 경우 "id" 사용  
            user-name-attribute: id
```