> 자원 자체를 사용하는 것 뿐만 아니라 사용이 끝났을 때 해제하는 것도 매우 중요한 일입니다. 

사용 후 반납해야 하는 자원을 try-with-resource로 쉽게 관리할 수 있습니다.  
try-with-resource는 autoCloseable (자동으로 닫히는) 인터페이스를 구현하는 리소스를 자동으로 닫아주는 역할을 합니다.  
try 블럭을 벗어나는 순간 자동으로 `close()` 메서드가 호출되는 방식으로 동작합니다.

try-with-resource가 없던 시절에는 직접 자원을 닫아줘야 했는데요,  
이런 방식은 아래의 단점이 있었습니다.
1. 코드가 복잡해집니다.
2. close 구문을 누락할 여지가 있습니다.
3. 예외가 발생할 경우 자원 반납을 하지 못 할 수 있습니다.

try-with-resource는 이런 문제들을 해결해줄 수 있는 좋은 수단입니다.
## Reference
https://tecoble.techcourse.co.kr/post/2021-04-26-try-with-resource/