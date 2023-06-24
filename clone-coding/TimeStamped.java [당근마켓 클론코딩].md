# TimeStamped.java [당근마켓 클론코딩]

### 개요
당근 마켓 클론 코딩을 진행하던 중, 코드 상 유의미하게 학습할 요소들이 있었기에 코드들을 하나 분석하며 모르는 부분들을 채워나갈 예정입니다. 이번 포스팅에서는 특정한 시각을 기록하기 위한 TimeStamp 클래스를 가져와봤는데, 이와 관련해서 작성된 @MapperSuperclass나 @EntityListeners 등을 알아보겠습니다.

### 코드
```java
import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.Column;
import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
// @EntityListeners(AuditingEntityListener.class) 제거해도 무방 
public class TimeStamped {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;
}
```

### @MappedSuperclass의 역할

JPA에서 사용되는 애노테이션으로 상속 관계에서 부모 클래스에 해당 어노테이션을 지정함으로써 부모 클래스의 속성들을 하위 엔티티 클래스에서 사용할 수 있도록 하는 역할을 합니다.

예를 들어, TimeStamped는 LocalDateTime을 가지고 있는데, 각각의 클래스는 TimeStamped를 상속함으로써 별도의 LocalDateTime 속성을 정의할 필요가 없는 것이죠.

+프로젝트 내에서 사용되는 예시

```java
public class Post extends TimeStamped 
public class ChatMessageStorage extends TimeStamped
```

### 🤔 근데 일반 상속 관계랑 차이가 뭐야?

우선 @MappedSuperClass는 상속 관계 매핑이 아닙니다. 해당 어노테이션이 붙은 클래스는 단지 공통의 속성들을 자식 클래스에게 제공하는 역할만을 합니다. 테이블과 매핑도 되지 않죠.

위 예제는 프로젝트 도중 나온 예시이고 조금 더 쉬운 예시로 설명해보겠습니다.

Team과 Member 엔티티가 존재한다고 가정합시다. 둘의 공통적인 속성은 무엇일까요? id와 name입니다. 우리는 공통되는 속성들을 한 번만 정의하고 싶습니다. 어떻게 해야할까요?

```java
@Data
@MapperSuperClass
class BaseEntity {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	Long id; 

	@Column(nullable = false)
	String name;
}
```

```java
@Entity
class Member extends BaseEntity {
							...
}

@Entity
class Team extends BaseEntity {
							...
}
```

이처럼 BaseEntity를 상속하면 깔끔하게 중복을 제거한 코드가 완성되었습니다.

### @EntityListeners(AuditingEntityListener.class)란?

@EntityListeners(AuditingEntityListener.class)는 JPA에서 사용되는 애노테이션으로, 엔티티의 생성 및 수정 시 자동으로 일부 필드 값을 설정해주는 기능인 Auditing을 구현하기 위해 사용됩니다.

예를 들어, 다음 코드를 보면 @LastModifiedDate 어노테이션을 통해 마지막으로 수정된 날짜를 저장하는 속성이 있습니다. 이는 @EntityListeners(AudidingEntityListener.class)와 같이 사용해야 합니다. 단, @CreatedDate은 위 어노테이션과 독립적으로 사용 가능합니다.

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @CreatedDate
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```