강의를 듣는 중 `build.gradle`의 다음의 설정을 추가하는 것을 보았습니다.
```kotlin
allOpen {  
    annotation("jakarta.persistence.Entity")  
    annotation("jakarta.persistence.MappedSuperclass")  
    annotation("jakarta.persistence.Embeddable")  
}  
  
noArg {  
    annotation("jakarta.persistence.Embeddable")  
}
```

*이 두 설정의 역할은 무엇일까요?*

우선 앞선 예시는 코틀린 플러그인 예시입니다.

**allOpen 플러그인**
Kotlin은 기본적으로 모든 클래스가 `final`이기 때문에, 상속이 불가능합니다. 그러나 JPA와 같은 프레임워크에서는 엔티티 클래스를 프록시(proxy)를 통해 상속하는 경우가 많습니다. `allOpen` 플러그인은 특정 애너테이션이 적용된 클래스가 `final`이 아니도록 설정하여, 이러한 프레임워크의 요구사항을 만족시킵니다.

**noArg 플러그인**
JPA와 같은 프레임워크에서는 종종 파라미터가 없는 생성자를 요구합니다. Kotlin에서는 기본적으로 명시적으로 생성자를 선언하지 않으면 파라미터가 없는 생성자가 자동으로 생성되지 않습니다. `noArg` 플러그인은 특정 애너테이션이 적용된 클래스에 대해 파라미터가 없는 생성자를 자동으로 추가합니다.

생소했던 이유는 아무래도 자바로 계속 개발을 해왔던 터라 코틀린의 플러그인에 대해 접할 기회가 없었기 때문입니다.

### 결론
Kotlin의 모든 클래스가 final인 점, 그리고 파라미터가 없는 생성자가 자동으로 생성되지 않는 특성이 JPA를 사용함에 있어서 제약조건이 되었고 이를 해결하기 위해 위 두 플러그인이 사용된 것입니다.