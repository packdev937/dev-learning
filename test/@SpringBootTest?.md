#### Layered Architecture
Layered Architecture 기반 Spring Boot 프로젝트라고 가정하고 진행해보겠습니다. 

![[LayeredArchitecture.png]]
그림으로 표현해본다면 다음과 같습니다.

우선 Domain은 대게 POJO로 이루어져 있어 별도의 `@SpringBootTest`가 필요하지 않습니다.
여기서 `@SpringBootTest` 가 필요한가에 대한 의미는 다음과 같습니다.
	***"SpringContext가 필요하는가"***

Domain을 제외한 나머지 계층들은 사실 SpringContext가 관리하는 Bean이 필요합니다. 

예를 들어, Service 계층의 비즈니스 로직을 테스트 하기 위해서는 Repository도 주입 받아야합니다. (데이터 베이스에 제대로 저장됐나 확인해야 하기 때문에)

계층 별 검증 사항은 다음과 같습니다.
- Domain : 일반 클래스와 동일하게 검증
- Service/Repository : 데이터 위주의 검증
- Controller : HTTP 위주의 검증 

#### 생성 테스트와 조회 테스트
테스트가 끝나면 공유 자원인 DB를 깨끗하게 클리어 해주어야 합니다. 
왜냐하면 여러 테스트는 Spring Context를 공유하기 때문입니다.

이 때, `@BeforeEach`혹은 `@AfterEach`를 사용합니다.

