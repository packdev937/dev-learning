## 1. 개요 
NGINX를 이용해 배포를 진행할 때 무작정 200을 반환하던 오류가 있었습니다. 어떤 값을 쓰던, 심지어 [domain 주소]/fewmfi032miodsf32 와 같은 쓰레기 값을 넣어도 200이 반환되었는데요, 사실 이는 Spring Security가 원인이었습니다. 

오늘 포스팅에서는 Spring Security가 왜 계속 200을 반환 했는지, 어떤게 문제였고 어떻게 해결했는지 알아보도록 하겠습니다. 

## 2. 트러블 슈팅
사실 이를 발견하기 까지는 매우 어려웠습니다. 레퍼런스가 현저히 부족했기 때문인데요, 결론적으로 말하면 스프링 시큐리티의 미숙함과 커스텀 필터에 대한 이해 부족으로 일어난 결과였습니다. 

앞서 다른 Spring Security 관련 글에서도 볼 수 있듯 저는 JwtAuthenticationFilter 라는 커스텀 필터를 사용해서 인증을 진행하고자 했습니다. 

💡 모르시는 분들을 위해서 
JwtAuthenticationFilter는 JWT Token에서 subject을 추출하여 유저를 식별할 수 있는 값을 얻고 해당 정보를 통해 Authentication 객체를 반환하는 필터입니다. 

처음에 작성했던 코드 중 일부분인데요 뭐가 문제인지 아시겠나요?
```
 String refreshToken = null;  
       try {  
           refreshToken = extractRefreshToken(request)  
			.filter(jwtTokenProvider::isTokenValid)  
			.orElse(null);  
		catch (Exception e){
			return;
		}
```

바로 예외가 터지면 return;을 해주는 점입니다.

doFilter는 FilterChain의 횟수만큼 시행되어야 합니다. 하지만 예외가 터지고 이를 잡는 catch문에서 바로 `return;` 을 사용하게되면 `doFilter()`는 당연히 실행되지 않습니다.

이 외에도 다양한 문제로 인해 무조건 200 반환 에러가 발생할 수 있다고 합니다. 그 리스트는 다음과 같습니다.

1. Filter 단에서 Request 나 Response 의 버퍼를 읽는 과정 속에 내용이 유실된 경우
    
2. 커스텀 필터 내 doFilter() 메소드를 실행하지 않은 경우 → 저는 이 경우에 속합니다.
    
3. 커스텀 필터에 조건문 등을 달아, doFilter() 호출전에 return 을 넣은 경우 → 2와 비슷합니다.
    
4. 하나의 커스텀 필터에 doFilter() 를 N번 호출한 경우

### Reference
- https://developer-ping9.tistory.com/236