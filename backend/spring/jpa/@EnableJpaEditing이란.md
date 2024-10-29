`@EnableJpaAuditing`은 Spring Data JPA에서 제공하는 기능으로, 엔티티(Entity) 클래스에 자동으로 생성 일시, 수정 일시 등을 기록할 수 있도록 해줍니다. 즉, `@CreatedDate`, `@LastModifiedDate`와 같은 어노테이션을 사용하여 엔티티의 생성 시점과 수정 시점을 자동으로 관리할 수 있게 해줍니다. 

### 주요 역할:
1. **엔티티의 생성 일시와 수정 일시 자동 기록**:
   - `@CreatedDate`: 엔티티가 처음 생성될 때 자동으로 그 시간을 기록합니다.
   - `@LastModifiedDate`: 엔티티가 수정될 때 자동으로 수정 시간을 기록합니다.

2. **작성자 및 수정자 정보 자동 기록** (`@CreatedBy`, `@LastModifiedBy`):
   - 엔티티의 작성자와 수정자 정보를 자동으로 저장하고 관리할 수 있습니다.
   - 이 경우 작성자와 수정자 정보를 담을 수 있는 보조 클래스를 추가로 구성해야 할 수 있습니다 (주로 Spring Security와 연동해서 사용).

### 예시:
```java
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    // Getter, Setter
}
```

`@EnableJpaAuditing`을 선언한 `JpaAuditingConfig` 클래스가 활성화되면, 엔티티에서 `@CreatedDate`나 `@LastModifiedDate`와 같은 어노테이션을 사용할 수 있게 됩니다. 

이렇게 하면 엔티티가 생성되거나 수정될 때 해당 필드들이 자동으로 채워집니다. 이는 일일이 시간을 기록하거나 수정하는 코드를 작성하지 않도록 도와줍니다.

### 핵심 개념:
- **Auditing**은 엔티티의 생성/수정 시간, 작성자, 수정자 등의 정보를 자동으로 관리하는 기능입니다.
- 이를 통해 애플리케이션이 데이터를 처리하는 흐름에서, 데이터를 언제, 누가, 어떻게 다뤘는지를 기록하는 로깅 역할을 수행할 수 있습니다.