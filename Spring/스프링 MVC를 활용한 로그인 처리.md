# 로그인 처리

### 개요

스프링 MVC 2편 로그인 처리를 보고 핵심 내용을 정리했습니다.

### 😌 로그인 로직을 어떻게 짤까요?

웹 사이트에 로그인 할 때를 떠올려봅시다. 어떤 과정이 있나요? 시스템 뒤에는 수많은 복잡한 로직이 있겠지만 단순하게는 아이디와 비밀번호를 입력하면 됩니다.

코드 또한 비슷합니다. 간단하게 과정은 다음과 같습니다.

1. 로그인 아이디와 비밀번호를 입력
2. 입력된 정보를 토대로 로그인 가능 여부 확인
3. 만약 실패한다면 다시 로그인 창으로 이동

코드를 함께 살펴보겠습니다.

```jsx
@GetMapping("/login")
public String loginForm(@ModelAttribute("loginForm" LoginForm form) {
	return "login/loginForm";
}

@PostMapping("/login")
public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult){
	if(bindingResult.hasError()){
		return "login/loginForm";
	}

	// 입력받은 아이디와 비밀번호를 통해 User를 찾는 과정
	Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

	if(loginMember == null) {
		bindingResult.reject("loginFail", "아이디, 비밀번호가 없습니다.");
		return "login/loginForm";
	}

	return "redirect:/"
}
	
```

늘 그렇듯 회원 가입이나 로그인 등 어떠한 입력을 받는 것은 @GetMapping과 @PostMapping을 함께 써주어야 합니다.

두 번째로 입력을 받는 과정에서 로그인한 객체를 못찾거나, 에러가 발생한 경우 다시 로그인 화면으로 이동합니다.

마지막으로 아무런 이상 없이 로그인 됐다면 홈으로 이동합니다.

로그인이 완료되면 로그인 정보가 남아있어야 겠죠. 어떻게 해야 할까요? 쿠키를 이용합니다.

### 🤔 왜 쿠키를 사용해야 하나요?

로그인의 상태를 유지하기 위해 사용합니다. 자세한 내용은 [쿠키](https://www.notion.so/3ecbf00ecb4e4d1ba2ee345792340545) 를 참고해주세요.

위 프로젝트에서는 다음과 같은 절차를 통해 쿠키를 저장합니다.

1. 클라이언트는 로그인 폼에서 로그인 아이디와 비밀번호를 입력합니다.
2. 서버는 `@PostMapping`으로 전달 된 로그인 아이디와 비밀번호 정보를 `Service`단의 메소드를 통해 확인합니다.
3. Service 단에서는 `Repository`단을 통해 해당 Id로 저장된 Member 엔티티가 있는지 확인하고, 있다면 비밀번호를 비교하여 로그인 가능 여부를 판단합니다
4. 로그인이 가능하다면 Set-Cookie 헤더를 통해 id 값을 넣어 보냅니다.
5. 해당 Cookie는 내부 쿠키 저장소에 저장되며, 후에 다시 페이지에 접속할 때 쿠키 정보를 확인함으로써 로그인 정보를 유지합니다.

쿠키는 또한 영속 쿠키와 세션 쿠키가 있습니다. 본 프로젝트에서는 브라우저 종료시 로그아웃이 되야하므로 세션 쿠키를 사용합니다.

### 😗 스프링에서는 세션 쿠키를 어떻게 생성하나요?

앞서 언급했다시피 Service 단에서 로그인 성공 여부를 판단합니다. 이 때 성공을 했다면 Session을 만들어줍니다.

```jsx
// Cookie에 매개변수를 넘길 때는 모두 String으로 넘겨야 합니다. 
Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));

// response에 쿠키를 넣어서 보내야 하므로 HttpServletResponse를 매개변수에 추가하였습니다. 
response.addCookie(idCookie);
```

+ 쿠키에 시간 정보를 주지 않으면 디폴트로 세션 쿠키가 됩니다.

### 😄 쿠키를 처리해보자

이제 쿠키 유무에 따라 처리를 달리 해야합니다.

이는 쿠키 정보를 확인해서 쿠키가 있다면 로그인한 화면으로, 없다면 홈 화면으로 이동하면 되겠죠? 코드는 다음과 같습니다.

```jsx
@GetMapping("/")
public String loginHome(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
	if(memberId = null) {
		return "home";
	}

	Member loginMember = memberRepository.findById(memberId);
	if(loginMember == null){
		return "home";
	}

	model.addAttribute("member", loginMember);
	return "loginHome";
}
```

위 코드를 보면 우선 @CookieValue 주석을 사용합니다. memberId로 매핑된 쿠키를 찾아주는데, 이 때 이 값이 필수는 아니기 때문에 required 속성을 false로 설정해줍니다. 필수가 아니라는 것은 즉 쿠키가 없는 사용자가 해당 경로를 요청할 수도 있다는 뜻입니다.

memberId가 null, 즉 없다면 홈 화면으로 보내줍니다. 이 때, else 문을 쓰지 않죠. 클린 코드의 법칙 중 하나 입니다!

또한, memberId가 있더라도 loginMember 객체가 없는 경우도 존재하는데요 이 때도 홈 화면으로 안내합니다.

따라서 쿠키의 memberId가 존재하고 해당 아이디로 조회했을 때의 멤버 엔티티가 정상적으로 존재한다면 로그인 이후의 화면으로 이동하겠네요!

### 🏃 로그인이 있다면 로그아웃도

로그인이 가능했다면 로그아웃도 있어야겠죠. 이는 간단하게 쿠키를 삭제하면 됩니다. 쿠키를 삭제하는 방법은 쿠키 종료 시간을 0으로 설정하면 됩니다. 이는 코드로 한번 살펴보겠습니다.

```jsx
@PostMapping("/logout")
public String logout(HttpServletResponse response) {
	Cookie cookie = new Cookie("memberId", null);
	cookie.setMexAge(0);
	response.addCookie(cookie);
	return "redirect:/";
}
```

쿠키를 없애는 로직은 다양하게 활용될 가능성이 높죠? 이는 리팩토링으로 해결해줍니다.

```jsx
@PostMapping("/logout")
public String logout(HttpServletResponse response) {
	expireCookie(response, "memberId");
	return "redirect:/";
}

private void expireCookie(HttpServletResponse response, String cookieName){
	Cookie cookie = new Cookie(cookieName, null);
	cookie.setMaxAge(0);
	response.addCookie(cookie);
}
```

### 🥲 저렇게 해도 안전할까요?

위 방식은 쿠키에 대한 개념을 간단하게 적용해서 만든 로그인/로그아웃 로직일 뿐 보안 상의 큰 문제가 존재합니다! 이를 자세히 알아보겠습니다.

첫 번째로 쿠키의 값은 임의로 변경할 수 있습니다. 방금 전의 예제에서 `F12 - 애플리케이션 - Cookies`에 들어가면 쿠키의 내용들을 확인할 수 있었죠. 그리고 로그인 로직은 단순히 쿠키의 값을 받아와서 해당 값을 토대로 회원을 불러옵니다.

즉, 만약 클라이언트가 임의로 쿠키의 값을 바꾼다면 해당 값을 아이디로 가진 다른 사용자의 정보로 로그인을 할 수 있는거죠!

두 번째로 쿠키에 보관된 정보는 **훔쳐**갈 수 있습니다. 실제로 쿠키는 웹 브라우저에 보관되고, 누구나 쉽게 확인할 수 있습니다. 만약 쿠키에 민감한 정보를 담고 있다면 이는 노출될 가능성이 매우 높은 것이죠.

또한 훔쳐간 쿠키를 통해 악용할 가능성이 매우 높습니다. 쿠키의 정보가 매우 민감한 정보 (카드 정보) 였다면 해당 정보를 다른 곳에 악용할 수도 있겠죠.

### 😱 그럼 어떻게 해야 하나요!

쿠키에 중요한 값은 노출하면 안됩니다. 사용자 별로 예측 불가능한 임의의 토큰 값을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식하도록 로직을 구성해야 합니다. 또한 서버에서 해당 토큰을 관리해야 합니다.

또한 해커가 임의의 값을 넣어도 토큰을 찾을 수 없도로 해야 합니다.

만약 이러한 보안 장치에도 불구하고 토큰을 훔쳐갔다면 해당 토큰의 만료시간을 짧게 유지함으로써 보안을 강화합니다. 토큰의 해킹이 의심된다면 강제로 제거할 수 있어야 합니다.

서버에 중요한 정보를 보관하고 연결을 유지하는 것을 **세션**이라고 합니다.

동작 과정은 다음과 같습니다.

1. 로그인 아이디와 비밀번호를 서버로 보내는 것 까지는 동일합니다.
2. 만약 해당 로그인 아이디로 조회되는 엔티티 정보가 있다면 해당 엔티티를 UUID라는 값을 생성해서 key와 value의 형태로 **세션 저장소**에 저장합니다.
3. 쿠키에 세션 값 (UUID) 값을 넣어 보내줍니다.
4. 웹 브라우저의 쿠키 저장소에는 임의로 만든 랜덤 세션 값을 가지고 있습니다.

임의의 값을 보냄으로써 엔티티와 관련된 어떠한 정보는 클라이언트에 전달하지 않는다는 것입니다.

또한, 사용자가 임의로 토큰 값을 변경하므로써 다른 사용자의 계정에 접속할 수 없습니다.

1. 이후에 세션 값을 Cookie에 실어 요청을 한다면 서버는 세션 저장소를 뒤져 해당 세션 값의 엔티티를 사용한다.

### 🤩 직접 세션 코드를 구현해보자

위 세션 과정을 통해 필요한 코드들을 살펴봅시다.

우선은 세션을 만들어야겠죠! 세션은 UUID라는 값을 생성해서 key와 value의 형태로 세션 저장소에 저장한다고 되어있습니다.

세션 저장소는 Map으로 구현하고 key는 UUID로 생성한 랜덤 값, value는 Object를 넣어주면 되겠네요! 또 해당 쿠키를 response를 통해 반환하면 끝이 납니다. 코드는 다음과 같습니다.

```jsx
public class SessionManager {
	public static final String SESSION_COOKIE_NAME = "mySessionID";
	private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

	public void createSession(Object value), HttpServletResponse response){
		String sessionId = UUID.randomUUID().toString();
    sessionStore.put(sessionId, value);

    Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
    response.addCookie(mySessionCookie);
}
```

세션을 저장하는 기능이 있다면 조회하는 기능도 있어야 겠죠? 조회는 어떻게 동작할까요? 우선 request에서 넘어온 쿠키의 이름과 ID 값이 있을 것 같습니다. Cookie를 모두 조회해서 그 중에 쿠키의 이름이 “mySessionId”인 것들을 찾고 해당 쿠키들 중 ID값을 얻어서 세션 저장소에 조회하면 엔티티가 나오겠네요.

말이 조금 복잡하지만 코드로 하나씩 살펴보면 어렵지 않습니다.

```jsx
		public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }
        return sessionStore.get(sessionCookie.getValue());
    }

    public Cookie findCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) {
            return null;
        }
        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
```

세션을 없애는건 어떻게 할까요? 세션 저장소에서 요청하는 세션의 정보를 삭제하면 되겠죠?

```jsx
		public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }
```

### 🤓 만들어진 Session을 사용해보자!

서블릿은 HttpSession을 제공합니다. 이는 직접 만든 세션과 유사하게 동작합니다.

기존에는 loginService.login()을 다 통과하면 정상적으로 로그인이 됐다고 판단하고 위에 정의한 createSession을 호출하였습니다.

이제는 다음과 같이 수행할 수 있습니다.

`login`

```jsx
// 로그인 아이디 패스워드 검증
// 로그인 성공 처리
HttpSession session = request.getSession();
session.addAttribute(SessionConst.LOGIN_MEMBER, loginMember);

return "redirect:/";
```

Session에 중복된 이름으로 값이 저장되는 것을 막기 위해 다음과 같이 클래스를 만들어줍니다.

```jsx
package hello.login.web;
  public class SessionConst {
      public static final String LOGIN_MEMBER = "loginMember";
}
```

로그아웃은 어떻게 할까요? 기존에는 로그아웃하고자 하는 엔티티를 불러와서 해당 엔티티의 세션 UUID 값을 찾고, 그 값을 통해 세션 스토어에 있는 값을 null로 만들었습니다.

`logout`

```jsx
			//세션을 삭제한다.
			HttpSession session = request.getSession(false); 
			if (session != null) {
          session.invalidate();
      }
      return "redirect:/";
```

하지만 getSession을 통해 (false에 주의해야 합니다. 만약에 false를 안하면 자동으로 세션 값이 만들어집니다) 세션을 불러온 뒤 invalidate()으로 삭제합니다.

**더 쉽게도 가능합니다**

```jsx
@GetMapping("/")
 public String homeLoginV3Spring(
          @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false)
					 Member loginMember,
	         Model model) {

	if (loginMember == null) {
          return "home";
  }

	model.addAttribute("member", loginMember); 
	return "loginHome";
}
```

만약 세션을 통해 Entity 객체가 확인되면 해당 객체는 loginMember에 매핑될 것입니다. 만약 없다면 null이 반환될 것이고, “home”으로 이동합니다.

있다면 Model 객체를 이용해서 회원 정보를 담은 뒤 loginHome으로 이동합니다.