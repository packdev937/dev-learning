### 1. 개요
본 포스팅은 최범균 님의 OOP 잘하려면을 듣고 정리한 내용입니다. 
### 객체지향 
객체 지향의 기본적인 키워드는 다음과 같습니다.
- **기능**
- 캡슐화
- 추상화 

### 그리고 기능 
기능을 잘 나눠야 객체 지향을 잘 할 수 있습니다. 따라서 기능을 분할하는 연습이 필요합니다. 

회원 가입이라는 기능이 있다고 가정해봅시다. 
해당 기능은 1) 중복 체크 2) member 테이블에 insert 3) 회원 가입 이메일 발송 으로 나누어질 수 있습니다. 

기능을 나눌 때 주의할 점은 어떻게 구현하는것 보다는 무엇을 하느냐에 집중한 기능 분리가 필요합니다.  

중복 체크 <-> member 테이블에서 id로 count
회원 저장 <-> member 테이블에 insert
가입 안내 통지 <-> 회원 가입 이메일 발송 

### 나눈 기능을 할당해보자 
기능을 다 나누었다면 이제는 하위 기능을 누가 할지 생각해봅시다. 
![[스크린샷 2024-03-29 오후 3.23.03.png]]

### 연습에 필요한 도구
![[스크린샷 2024-03-29 오후 3.26.21.png]]
### Reference 
- https://www.youtube.com/watch?v=UbEP86HbJkI