멀티 모듈에서 Properties 클래스를 사용하는 빈을 주입받아 사용할 때 양쪽 모듈 application.yml에 동일한 내용을 적어주어야 합니다. 

예를 들어, JwtProperties를 사용하기 위해 jwt 관련 설정들을 application.yml에 작성했다고 가정해보겠습니다.

`JwtProperties.java`는 domain 모듈에 작성되었으며 `JwtProvider.java`가 의존성 주입 하여 사용합니다.

중요한건 `JwtProvider.java`는 api-server 모듈에서도 의존성을 주입받아 사용한다는 것입니다. 이 때, `JwtProvider`는 `JwtProperties`를 주입 받으므로 관련 내용이 api-server 모듈의 application.yml에도 작성되어야 합니다. 