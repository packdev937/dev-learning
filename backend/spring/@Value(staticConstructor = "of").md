### 1. 개요
백견이 불어일타, 코드로 해당 어노테이션을 먼저 보여드리겠습니다. 
```java
@Getter  
@Value(staticConstructor = "of")  
public class Id {  
  
    private final Long value;  
}
```

### 2. 환경 설정 
우선 @Value를 사용하기 위해서는 lombok 의존성을 추가해야 합니다. 

```java
dependencies {
	compileOnly 'org.projectlombok:lombok'  
	annotationProcessor 'org.projectlombok:lombok'
}
```

해당 속성을 사용할 경우 클래스 내 모든 필드가 Private 메소드로 생성되고 static한 생성자를 생성해줍니다.
주의할 점은 해당 속성을 사용하게 되면 정적 팩토리 메소드는 하나만 존재할 수 있습니다. 