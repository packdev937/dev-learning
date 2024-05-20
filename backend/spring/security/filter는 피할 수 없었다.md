## 1. 개요
늘 그렇듯, 프로젝트를 처음 시작하면 가장 먼저 설정하는 부분은 인증/보안과 관련된 부분입니다. 프로젝트 마다 이를 구현하는 방식은 다르지만 저는 Spring Security를 사용하는 것을 선호합니다.

오늘 포스팅에서는 Spring Security를 사용하며 발생했던 `permitAll()`과 `filter`와 관련된 오해를 풀고자 합니다.

## 2. 문제가 발생한 부분
Spring Security의 주된 사용 목표 중 하나는 접근 권한이 필요 있는 경로와 없는 경로를 구분하는 것입니다.

```sql
	  @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
							.requestMatchers(RESOURCE_LIST).permitAll()
							.anyRequest().authenticated())
```

보통은 다음과 같은 설정을 통해 접근 권한을 제어합니다.

이번 프로젝트에서는 회원 가입을 하거나, 스웨거 문서를 확인하는 등의 경로 이외에는 모두 검증이 필요했습니다. 따라서 검증이 필요 없는 경로에 대해서 `.permitAll()` 을 설정해주었고 순조로운 시작을 예상했습니다.

하지만 모든 프로젝트가 그렇듯 문제가 어김없이 발생 하였는데, 권한을 허용한 웹 사이트에서 계속 403 에러가 반환되는 것이었습니다.

```sql
.addFilterBefore(
		new JwtAuthenticationFilter(jwtTokenProvider, loadUserPort, saveUserPort, authenticationService),
		UsernamePasswordAuthenticationFilter.class);
```

`filterChain()` 메소드의 하단에는 위와 같은 설정이 포함되어 있습니다. Filter를 등록하는 과정인데요, 저는 `UsernamePasswordAuthenticationFilter` 이전에 커스텀한 Filter인 `JwtAuthenticationFilter`를 등록해주었습니다. 이렇게 설정함으로서 권한이 필요한 경로에 대해서는 해당 Filter를 통과하여 권한 유무를 판단합니다.

## 3. Filter와 .permitAll()은 무관
하지만 이는 크나큰 착각이었는데 디버깅 포인트를 통해 확인해보니 **permit** 여부와 상관 없이 Filter를 통과하는 것이었습니다. `JwtAuthenticationFilter`의 역할은 헤더에 유효한 `JWT Token`이 포함됐는지 검증하는 것이었기에 Filter로 진입하는 순간 **permit** 여부는 상관 없다고 생각했습니다.

### 3-1. 진정한 permit()의 역할
그렇다면 403 에러를 피할 수 없는 건가요? 아닙니다. permit()은 filter를 무시하는 역할이 아닙니다.

`JwtAuthenticationFilter` 혹은 `UsernamePasswordAuthenticationFilter`는 헤더로 오는 정보를 통해 유효성을 검증하고 `Authentication` 객체를 생성합니다.

생성된 `Authentication` 객체는 후에 **Controller** 혹은 다른 **Util 단**에서 사용자를 식별하는 역할을 합니다. 여기서 주목해야될 점은 `permit()` 그리고 `authenticated()` 는 Authentication 객체 생성 여부에 따라 경로를 허용합니다.

즉, `permit()`이 붙어 있는 경로에 대해서는 **Authentication** 객체가 생성되지 않아도 허용하며, 반대로 `authenticated()` 가 붙어 있는 경로에 대해서는 **Authentication** 객체가 생성되지 않았다면 예외를 발생합니다.

## 4. 결론
결론적으로 우리는 헤더에 아무 값도 들어 있지 않을 때의 Filter를 잘 정의해야 합니다. Authentication 객체가 생성되지 않아도 permit()이 붙어있으면 문제 없지만, 정작 정보가 없기 때문에 발생하는 예외에 대해서는 문제가 발생할 여부가 있기 때문입니다.

### Reference
- [Velog Blog (1)](https://velog.io/@dangddoong/TIL-22.12.29-permitAll%ED%95%98%EB%A9%B4-SecurityFilterChain%EC%9D%84-%EC%95%88%ED%83%80%EB%8A%94%EC%A4%84-%EC%95%8C%EC%95%98%EC%A7%80-%EB%AD%90%EC%95%BC)
- [Java Docs](https://docs.spring.io/spring-security/site/docs/4.2.x/apidocs/org/springframework/security/config/annotation/web/configurers/ExpressionUrlAuthorizationConfigurer.ExpressionInterceptUrlRegistry.html) 
- [Notion Blog (1)](https://catsbi.oopy.io/c0a4f395-24b2-44e5-8eeb-275d19e2a536)

