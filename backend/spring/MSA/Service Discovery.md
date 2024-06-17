## 1. Service Discovery란
> 외부에서 다른 서비스들이 마이크로 서비스를 검색하기 위해서 사용되는 개념입니다. 

MSA로 서비스를 구축하게 되면 수 많은 서버가 생성됩니다. 각각의 서버는 IP를 가지고 있습니다. 문제는 클라우드 환경에서 각 서버의 IP가 동적으로 바뀌게 되고, 이를 수동으로 추적하기는 매우 어렵습니다. 따라서 Service Discovery에 서버를 등록하고 검색하여 마이크로 서비스 간 통신을 가능하게 해줍니다. 

Spring에서는 대표적으로 Spring Eureka를 사용하여 Service Discovery 작업을 진행할 수 있습니다.

## 2. hands-on
### 2-1. Service Discovery 
실습을 한번 해보겠습니다. 
스프링 프로젝트를 하나 만들고 `build.gradle`에 다음 의존성을 추가해줍니다. 
`build.gradle`
```gradle
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
```

`SpringMsaApplication.java`
```
@SpringBootApplication
@EnableEurekaServer
public class SpringMsaApplication {
	... 
}
```

Eureka 서버 역할을 하기 위해서는 어플리케이션을 서버의 자격으로서 등록을 해주어야 합니다. 
위의 `@EnableEurekaServer` 어노테이션을 붙여주게 된다면 Spring Boot가 초기 가동되면서 해당 정보를 읽고 Service Discvoery 로서 어플리케이션을 가동합니다. 

`application.yml`
```yaml
server:  
  port: 8761  
  
spring:  
  application:  
    name: spring-msa  
  
eureka:  
  client:  
    register-with-eureka: false  
    fetch-registry: false
```
기본적인 .yml 설정파일의 구성이며 `register-with-eureka`와 `fetch-registry`를 `false`로 설정함으로써 자기 자신을 Service Discovery에 등록하는 것을 방지하였습니다.

이 상태에서 어플리케이션 실행 후 http://localhost:8761 로 접속하면 다음과 같은 화면을 확인할 수 있습니다. 
![[스크린샷 2024-06-17 오전 9.39.21.png]]
위 화면은 대시보드로서 등록된 인스턴스와 관련된 정보들을 확인할 수 있습니다.

### 2-2. Client 서버 
다음으로 Client 서버를 만들어보겠습니다. 
새로운 스프링 프로젝트를 생성하고 의존성은 다음을 추가해줍니다.
```gradle
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'  
implementation 'org.springframework.boot:spring-boot-starter-web'
```

그리고 Application 클래스에 `@EnableDisvoceryClient`를 추가해줍니다. 
`UserServiceApplication.java`
```java
@SpringBootApplication  
@EnableDiscoveryClient  
public class UserServiceApplication {
	...
}
```

이후 두 프로젝트를 동시에 실행하면 Service Discovery에 user-service가 등록된 것을 확인할 수 있습니다.
![[스크린샷 2024-06-17 오후 1.05.07.png]]

#### 2-2-1. 다른 포트 번호로 실행하기
상단의 Edit Configurations를 클릭하고 
![[스크린샷 2024-06-17 오후 1.22.21.png]]

add VM Option에 `-Dserver.port = 9002`를 넣어주면, 다른 포트번호로 똑같은 서버가 실행되게 됩니다.
![[스크린샷 2024-06-17 오후 1.22.08.png]]

결과는 다음과 같습니다. 
![[스크린샷 2024-06-17 오후 1.23.18.png]]

### 로드 밸런서 
매번 인스턴스를 가동할 때 마다 포트 번호를 변경하는 것은 귀찮은 작업입니다. 우리는 스프링 부트에서 지원해주는 랜덤 포트 기능을 사용하여 이를 간편화 할 수 있습니다. 

`application.yml`
```yaml
server:  
  port: 0  
  
spring:  
  application:  
    name: user-service  
  
eureka:  
  instance:  
    instance-id: ${spring.cloud.client.hostname}:${spring.application.instance_id:${random.value}}  
  client:  
    fetch-registry: true  
    register-with-eureka: true  
    service-url:  
      defaultZone: http://127.0.0.1:8761/eureka/
```