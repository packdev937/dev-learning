우선 BaseEntity가 왜 필요한지 부터 정의해봐야 합니다. 
보통 공통의 속성이 사용될 때 BaseEntity를 상속해줌으로써 코드를 더 간결하게 유지합니다. 

예제 코드를 보면서 살펴보도록 합시다. 
```java
@EntityListeners(AuditingEntityListener.class)  
@Getter  
@MappedSuperclass  
public class BaseEntity {  
  
    @CreatedDate  
    @Column(updatable = false, name = "created_at")  
    private LocalDateTime createdAt;  
  
    @LastModifiedDate  
    @Column(name = "updated_at")  
    private LocalDateTime updatedAt;  
}
```

#### EntityListeners(AuditingEntityListener.class)
엔티티가 생성되거나 수정될 때 자동으로 특정 필드를 업데이트 합니다. 이를 통해 데이터의 일관성을 유지하고 수동으로 이러한 값을 설정하는 번거로움을 없애줍니다. 

또한, BaseEntity에 해당 어노테이션을 사용함으로써 이를 상속받는 모든 엔티티들이 자동으로 audit 기능을 활용할 수 있께 됩니다. 

#### MapperSuperclass
BaseEntity는 각종 Entity에 상속을 진행합니다. 그리고 Entity 클래스는 Entity 클래스만 상속을 받을 수 있습니다. 하지만 BaseEntity는 Entity라기에는 다소 모호한 감이 있습니다. 

*"Entity 클래스에서 일반 클래스를 상속받기 위해서는 해당 어노테이션을 작성합니다."*

#### CreatedDate
Entity가 생성되어 저장될 때 시간이 자동으로 저장됩니다. 

#### LastModifiedDate
Entity이 값을 변경할 때 시간이 자동으로 저장됩니다. 

추가적으로 BaseEntity를 사용한다면 @EnableJpaAuditing에 추가해줘야 합니다.
```java
@SpringBootApplication  
@EnableJpaAuditing  
public class SunshineServerApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(SunshineServerApplication.class, args);  
    }  
  
}
```

이는 Auditing 활성화를 위해 필요한 작업입니다.

### Reference
- https://velog.io/@vpdls1511/Spring%EC%97%90%EC%84%9C-BaseEntity%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0