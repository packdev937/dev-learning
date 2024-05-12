프로젝트 개발 중 다음과 같은 경고 메세지를 받았습니다. @Builder.Default를 왜 붙이라고 경고하는 걸까요?

![[스크린샷 2024-05-12 오후 5.17.28.png]]

Lombok 라이브러리의 `@Builder` 어노테이션을 사용할 때 해당 경고 메시지가 표시되는 이유는 `@Builder`가 클래스 필드를 초기화하는 방식과 관련이 있습니다. Lombok의 `@Builder`는 객체를 불변성을 가진 객체로 만드는 데 유용하며, 객체의 인스턴스를 생성할 때 명시적으로 값들을 설정할 수 있게 해줍니다.

여기서 주의할 점은 Lombok이 필드 초기화를 무시하고 `@Builder`를 사용해서 객체를 생성할 때 해당 필드(`archives`)를 기본적으로 `null`로 설정한다는 것입니다. 따라서 초기화식(`new ArrayList<>()`)은 빌더를 통해 생성된 객체에 반영되지 않습니다. 이는 런타임에서 `NullPointerException`을 유발할 수 있습니다.

Lombok은 이 문제를 해결하기 위해 두 가지 방법을 제안합니다:

1. **`@Builder.Default` 사용하기**: 필드의 초기화 식을 빌더에 포함된 객체의 기본값으로 사용하고 싶을 때 `@Builder.Default`를 사용합니다. 이렇게 하면, 빌더를 통해 명시적으로 값이 설정되지 않았을 때, 초기화 식(`new ArrayList<>()`)이 기본값으로 사용됩니다.
    
    예를 들어:
    
    java
    
    Copy code
    
    `@Builder.Default public List<ArchiveJpaEntity> archives = new ArrayList<>();`
    
2. **필드를 `final`로 만들기**: 필드가 생성자를 통해 한 번만 초기화되어야 하고 빌더를 통해 설정할 수 없어야 할 때 `final` 키워드를 사용합니다. 이 경우, 필드는 생성 시에만 값을 할당받고, 빌더를 사용하여 해당 필드를 변경할 수 없습니다.
    
    예를 들어:
    
    java
    
    Copy code
    
    `public final List<ArchiveJpaEntity> archives = new ArrayList<>();`
    

경고 메시지에서는 두 가지 옵션을 제시하고 있으니, 필드 `archives`가 빌드 과정에서 기본적으로 빈 리스트를 가져야 한다면 `@Builder.Default` 어노테이션을 추가하는 것이 적절할 것입니다. 이렇게 하면 객체가 생성될 때 `archives` 필드가 `null`이 아닌 빈 리스트로 초기화되어 안전하게 사용할 수 있습니다.