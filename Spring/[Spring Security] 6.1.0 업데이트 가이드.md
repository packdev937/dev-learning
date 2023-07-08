# [Spring Security] 6.1.0 업데이트 가이드

### 개요

Spring Security 6.1.0 이후로, 인터넷 강의나 블로그에서 자주 보던 예제 코드들이 deprecated되었습니다. 이 변경에 대한 혼란을 방지하고자, 이 글에서는 기존 코드와 새롭게 업데이트된 코드를 비교하며 차이점을 정리해 보고자 합니다.

> 🤣 예시를 보면 아시겠지만 코드의 패턴들이 매우 유사합니다. 의도는 기존에 있던 `and()`를 제거하여 가독성을 높이고 관련성 있는 속성 끼리 묶고자 한 느낌입니다.
>

### csrf()

**기존 방식**

```java
http.csrf().disable();
```

**업데이트된 방식**

```java
http
	.csrf((csrf) -> csrf.disable());
```

### authorizeHttpRequest()

**기존 방식**

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http
		.authorizeHttpRequests()
			.requestMatchers("/**").hasRole("USER")
			.and()
		.formLogin();
	return http.build();
}
```

**업데이트된 방식**

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http
		.authorizeHttpRequests((authorizeHttpRequests) ->
			authorizeHttpRequests
				.requestMatchers("/**").hasRole("USER")
		)
		.formLogin(withDefaults());
	return http.build();
}
```

😮 **requestMatcher 사용 시 주의 사항**

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http
		.authorizeHttpRequests((authorizeHttpRequests) ->
			authorizeHttpRequests
				.requestMatchers("/**").hasRole("USER")
				.requestMatchers("/admin/**").hasRole("ADMIN")
		);
	return http.build();
}
```

위와 같은 코드는 조심해야 합니다. 모든 요청이 “/**”에서 처리되므로, 두 번째 매핑은 도달할 수 없습니다.

### formLogin()

**기존 방식**

```java
http.formLogin()
	.loginPage("/login.html")   				// 사용자 정의 로그인 페이지
	.defaultSuccessUrl("/home")					// 로그인 성공 후 이동 페이지
	.failureUrl("/login.html?error=true")		// 로그인 실패 후 이동 페이지
	.usernameParameter("username")				// 아이디 파라미터명 설정
	.passwordParameter("password")				// 패스워드 파라미터명 설정
	.loginProcessingUrl("/login")				// 로그인 Form Action Url
	.successHandler(loginSuccessHandler())		// 로그인 성공 후 핸들러
	.failureHandler(loginFailureHandler())		// 로그인 실패 후 핸들러
```

**업데이트된 방식**

```java
.formLogin((formLogin) ->
	formLogin
		.usernameParameter("username")
		.passwordParameter("password")
		.loginPage("/authentication/login")
		.failureUrl("/authentication/login?failed")
		.loginProcessingUrl("/authentication/login/process")
	);
```

### logout()

**기존 방식**

```java
http.logout()						// 로그아웃 처리
	.logoutUrl(＂/logout＂)				// 로그아웃 처리 URL
	.logoutSuccessUrl(＂/login＂)			// 로그아웃 성공 후 이동페이지
	.deleteCookies(＂JSESSIONID“, ＂remember-me＂) 	// 로그아웃 후 쿠키 삭제
	.addLogoutHandler(logoutHandler())		 // 로그아웃 핸들러
	.logoutSuccessHandler(logoutSuccessHandler()) 	// 로그아웃 성공 후 핸들러
```

**업데이트된 방식**

```java
http.logout(log -> log
	.logoutUrl("/logout")
	.logoutSuccessUrl("/login")
	.deleteCookies(＂JSESSIONID“, ＂remember-me＂) 	// 로그아웃 후 쿠키 삭제
	.addLogoutHandler(logoutHandler())		 // 로그아웃 핸들러
	.logoutSuccessHandler(logoutSuccessHandler()) 	// 로그아웃 성공 후 핸들러
```

### remeberMe()

**기존 방식**

```java
http.rememberMe()
		.rememberMeParameter(“remember”) // 기본 파라미터명은 remember-me
    .tokenValiditySeconds(3600) // Default 는 14일
    .alwaysRemember(true) // 리멤버 미 기능이 활성화되지 않아도 항상 실행
    .userDetailsService(userDetailsService)
```

**업데이트된 방식**

```java
http.rememberMe(r ->
			r.rememberMeParameter(“remember”)
			r.tokenValiditySeconds(3600)
			r.alwaysRemember(true))
```

### 참조
[[Spring Security] 기본 API 이해](https://velog.io/@wogh126/Spring-Security-기본-API-이해)

[[Spring] Spring Boot 3.1.0, Spring 6.1, Spring Security 6.1 : Security Config deprecated](https://velog.io/@jmjmjmz732002/Spring-Spring-Boot-3.1.0-Spring-6.1-Spring-Security-6.1-Security-Config-deprecated)