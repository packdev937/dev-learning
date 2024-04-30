우선 카카오 개발자 페이지 [앱 등록](https://developers.kakao.com/console/app)을 진행해야 합니다. 

앱 등록하면 다음과 같이 앱 키가 주어지는데, 현 시점에서 저희가 사용할 정보는 REST API 키 입니다.
![[스크린샷 2024-03-31 오후 9.22.00.png|500]]

좌측의 카카오 로그인 탭으로 이동하여 카카오 로그인을 활성화 해줍니다. 
![[스크린샷 2024-03-31 오후 9.29.10.png|500]]

다음으로 Redirect URI를 등록해줍니다. Redirect URI는 사용자가 로그인을 수행했을 때 발급되는 코드를 반환하는 페이지라고 생각하시면 됩니다. 

왜 리다이렉션 되는지 그 개념이 헷갈리신다면 OAuth의 흐름을 잘 파악하고 있어야 합니다. 
![[스크린샷 2024-03-31 오후 9.31.56.png]]

OAuth의 작동 방식은 다음과 같습니다.

1. 사용자는 서버에 로그인을 요청을 합니다.
2. 서버는 사용자에게 로그인 페이지를 응답합니다.
3. 사용자는 카카오 로그인을 클릭하여 소셜 로그인을 통한 로그인을 하고자 합니다.
4. 사용자는 인증 서버에 로그인합니다.
5. 사용자가 로그인에 성공하면, 인증 서버는 클라이언트 애플리케이션에 대한 인증 코드를 발급합니다.
6. 클라이언트 애플리케이션은 발급받은 인증 코드와 함께 인증 서버에 액세스 토큰을 요청합니다.
7. 인증 서버는 클라이언트 애플리케이션에 액세스 토큰을 발급합니다.
8. 클라이언트 애플리케이션은 액세스 토큰을 사용하여 보호된 리소스에 접근합니다.

이러한 방식으로 OAuth는 클라이언트 애플리케이션이 사용자의 비밀번호를 알 필요 없이 인증 및 인가를 처리할 수 있게 해줍니다. 또한, 사용자는 자신의 로그인 정보를 안전한 인증 서버와 공유하고, 클라이언트 애플리케이션은 사용자의 리소스에 대한 권한을 부여 받을 수 있습니다. 

다음으로 Redirect URI를 등록해줍니다.
![[스크린샷 2024-03-31 오후 10.00.09.png]]

아까 발급했던 REST API Key와 Redirect URI를 넣고 다음 URL을 실행해봅시다.
```bash
kauth.kakao.com/oauth/authorize?client_id=98e0050b4553c0f6dd346cf67dd612fd&redirect_uri=https://packdev937.site/oauth/kakao&response_type=code
```

`OAuthController.java`
``` java
@RestController  
public class OAuthController {  
  
    @GetMapping("/oauth/kakao")  
    public String kakao(@RequestParam("code") String code) {  
        return code;  
    }  
}
```

그러면 쿼리 파라미터로 코드가 날라오는 것을 확인할 수 있습니다. 
![[스크린샷 2024-03-31 오후 10.08.16.png]]

이제는 해당 인가 코드를 통해 토큰을 발급 받아야 합니다. 
![[스크린샷 2024-03-31 오후 10.13.09.png|700]]

POST 요청시 작성해야 하는 Body는 다음과 같습니다. 

`grant_type`은 authorization_code로 고정합니다. 
`client_id`는 등록 시 발급 받았던 REST API Key 입니다. 
`redirect_uri`는 인가 코드를 발급 받았던 당시 사용했던 URI를 그대로 입력합니다. 
`code` 또한 전달 받았던 인가 코드를 그대로 사용합니다.
`client_secret`은 토큰 발급 시 보안을 강화하기 위해 추가적으로 확인하는 코드입니다. 별도의 설정을 했다면 추가해줍시다. 

몇몇 코드를 추가해서 다시 구현해보겠습니다.

`OAuthController.java`
```java
@RestController  
@RequiredArgsConstructor  
public class OAuthController {  
  
    private final OAuthService oAuthService;  
  
    @GetMapping("/oauth/kakao")  
    public ResponseEntity<String> kakao(@RequestParam("code") String code) {  
        return ResponseEntity.ok(oAuthService.kakaoLogin(code));  
    }  
}
```

`OAuthService.java`
```java
@Service  
public class OAuthService {  
  
    private final String kakaoBaseUrl = "https://kauth.kakao.com";  
    private final String kakaoClientId = "98e0050b4553c0f6dd346cf67dd612fd";  
    private final String kakaoRedirectUri = "http://localhost:8080/oauth/kakao";  
  
    public String kakaoLogin(String code) {  
        WebClient webClient = WebClient.builder()  
            .baseUrl(kakaoBaseUrl)  
            .build();  
  
        KakaoOAuthResponse token = webClient.post().uri("/oauth/token")  
            .contentType(MediaType.APPLICATION_FORM_URLENCODED)  
            .body(  
                BodyInserters.fromFormData("grant_type", "authorization_code")  
                    .with("client_id", kakaoClientId)  
                    .with("redirect_uri", kakaoRedirectUri)  
                    .with("code", code))  
            .retrieve()  
            .bodyToMono(KakaoOAuthResponse.class)  
            .block();  
  
        return token.access_token();  
    }  
}
```

`KakaoOAuthResponse`
```java
public record KakaoOAuthResponse(String token_type, String access_token, String id_token, Integer expires_in, String refresh_token, String scope) {  

}
```

추가적으로 엑세스 토큰과 함께 반환되는 아이디 토큰을 통해 유저의 정보를 추출할 수 있습니다. 
단, 아이디 토큰을 파싱하기 위해서는 공개키가 필요합니다. 공개키를 발급 받는 방법은 [다음 문서](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#oidc-find-public-key)에서 확인 가능합니다. 

`https://kauth.kakao.com/.well-known/jwks.json` 해당 URI을 호출하면 공개키 리스트가 전달됩니다. 


### /v2/user/me

WebClient를 사용해 email을 불러오는 도중 계속 null 값이 들어갔습니다. 로그를 찍어보니 email은 kakao_account 안에 있는 자식 필드였습니다. 

```json
{"id":3415939150,
 "connected_at":"2024-03-31T13:50:28Z",
 "kakao_account": {"has_email":true,
	"email_needs_agreement":false,
	"is_email_valid":true,
	"is_email_verified":true,
	"email":"roy.pack.gun.woo@gmail.com"
	}
}
```

따라서 이를 다음과 같이 해결했습니다.
```java
@Override  
public OAuthDetailResponse retrieveOAuthDetail(String accessToken) {  
  
    return webClient.post()  
        .uri(uriBuilder -> uriBuilder.path("/v2/user/me")  
            .queryParam("property_keys", "[\"kakao_account.email\"]")  
            .build())  
        .header("Authorization", "Bearer " + accessToken)  
        .retrieve()  
        .bodyToMono(JsonNode.class)  
        .doOnNext(jsonNode -> log.info("Received JsonNode: " + jsonNode)) // JsonNode 확인  
        .map(jsonNode -> new OAuthDetailResponse(jsonNode.get("kakao_account").get("email").asText()))  
        .block();  
}
```

## Reference 
- https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#req-user-info
- https://devnm.tistory.com/35


kauth.kakao.com/oauth/authorize?client_id=98e0050b4553c0f6dd346cf67dd612fd&redirect_uri=http://localhost:9000/oauth/kakao&response_type=code