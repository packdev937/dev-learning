제가 현재 만들고 있는 서비스의 흐름은 다음과 같습니다. 
1. 구글 로그인을 진행한다.
2. 구글 로그인으로 생성된 계정이 있다면 해당 계정의 홈으로 이동한다.
3. 구글 로그인으로 생성된 계정이 없다면 생성 프로세스를 진행한다.

인터넷에는 OAuth를 활용하여 로그인을 구현할 수 있는 방법이 많이 나와있었고, 저 또한 구현 경험이 있어 크게 어렵지는 않을 것 같았습니다. 

하지만 문제는 기존의 경험과 다르게 **프론트와 백을 분리해서 생각**해야 했습니다.

먼저 간단하게 OAuth 2.0 프로세스를 확인해보겠습니다.
#### OAuth 2.0 프로세스 
![[OAuth 2.0 프로세스.png]]

앞서 말씀드렸듯 현재 진행중인 서비스는 **프론트와 백을 분리해서 개발**을 진행해야 됐는데, 위 프로세스에서 어디부터 어디까지 프론트가 맡고 어디까지 백인지 맡을 지 명확하게 구분할 필요가 있었습니다.

결론부터 말씀 드리면 Authorization Code를 발급받는 것을 프론트가, 해당 코드를 받아 Access Token을 발급 받는 것을 백엔드가 맡는 것으로 진행을 결정했습니다. 

이유는 다음과 같습니다. 
OAuth 인증 프로세스에서는 클라이언트를 인증하기 위해 `clientSecret` 변수를 사용합니다. 이는 안전하게 보관되어야 하는데요, 프론트엔드 코드는 클라이언트에서 실행되기 때문에 해당 코드가 완전하게 노출될 수 있습니다. 

### 하지만
[관련 글](https://velog.io/@iamtaehoon/OAuth-%EC%9D%B8%EC%A6%9D-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-Backend-API-%EA%B4%80%EC%A0%90%EC%97%90%EC%84%9C-%EC%9E%91%EC%84%B1%EC%A4%91)을 읽다 보니 아쉬운 점도 존재했습니다.
프론트와 백에서의 역할을 분리하다 보니 AuthorizationUri, clientId, redirectUri, scope에 관한 부분은 프론트가 clientSecret 관련한 부분은 백엔드 서버가 갖고 있다는 것입니다. 

따라서 인증 정책이 바뀐다면 프론트와 백 둘다 수정이 필요합니다. 

### 구현
빠른 시일 내에 완료해야 하는 것이 목표이기 때문에 이번 서비스에서는 프론트와 백에서 분리해서 작업을 하는 방식으로 진행해보겠습니다.

우선 사용 되는 **End Point**들을 알아보겠습니다. 

**Authorization Code 요청**
```
https://accounts.google.com/o/oauth2/v2/auth?scope=https%3A//www.googleapis.com/auth/drive.metadata.readonly&
 include_granted_scopes=true&
 response_type=token&
 state=state_parameter_passthrough_value&
 redirect_uri=https%3A//oauth2.example.com/code&
 client_id=client_id
```

이 때 redirect url은 서버 측의 


**Access Token 요청**
```
https://oauth2.googleapis.com/token?code=4/P7q7W91a-oMsCeLvIaQm6bTrgtp7&
client_id=your_client_id&
client_secret=your_client_secret&
redirect_uri=https%3A//oauth2.example.com/code&
grant_type=authorization_code
```

**Access Token 응답** 
```
{  
	"access_token": "1/fFAGRNJru1FTz70BzhT3Zg", 
	"expires_in": 3920,  "token_type": "Bearer",  
	"scope": "https://www.googleapis.com/auth/drive.metadata.readonly", 
	"refresh_token": "1//xEoDL4iW3cxlI7yDbSRFYNG01kVKM2C-259HOF2aQbI"  
}
```

**Refresh Token 으로 요청**
```
POST /token HTTP/1.1
Host: oauth2.googleapis.com
Content-Type: application/x-www-form-urlencoded

client_id=your_client_id&
client_secret=your_client_secret&
refresh_token=refresh_token&
grant_type=refresh_token
```

흐름 정리
post /login?auth_code={auth_code}로 AccessToken과 RefreshToken을 발급받음
이 때, AccessToken을 사용해서 UserInfo를 얻고, UserInfo에서 이메일을 추출함 

해당 이메일로 findUserByEmail 했는데 null 이면 불리안 값 false, 아니면 true를 access token과 함께 줌

만약에 불리안 값이 false 면 온보딩 프로세스를 진행, 해당 정보들을 가지고 유저를 생성

null 이면, auth code 얻을 때 받았던 정보들을 가지고 온보딩 프로세스 진행 후 같이 가입 
-> 찾아보니 email 등의 정보는 accessToken으로 요청한듯
-> 따라서 /create도 accessToken으로 요청해야 되고 거기서 email 추출해서 빌드해야될듯?

가입 후 -> access Token을 가지고 /home 요청을 하면 바로 퀘스트를 받을 수 있게 진행

참고로 access Token은 후에 JWT Token으로 
Spring Security 사용 안하고 해도 될듯?

토요일 -> OAuth 마무리 및 기타 API 수정 
일요일 -> 최종 테스트 및 S3 작업 들어가기 



`https://www.googleapis.com/auth/userinfo.email?access_token=access_token`


### Reference
https://blog.naver.com/nan17a/222182983858 

### 구글 인증 승인
![[스크린샷 2024-02-12 오전 11.51.13.png]]