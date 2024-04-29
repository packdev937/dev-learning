서비스의 현재 상황은 다음과 같습니다. 
- Spring Security를 사용하지 않음
- 따라서 CustomUserDetails도 구현하지 않음
- Google OAuth 로그인만 지원함 

이 때, 회원과 관련된 정보를 어떤 식별자를 통해서 조회할까요? 

우선 생각해본건 구글 로그인 진행 시 access token을 발급한다는 점입니다. 
access token과 함께 유저 정보 (sub, email) 등을 반환해주면 해당 정보를 식별자로서 사용할 수 있지 않을까요?