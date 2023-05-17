# 도메인 분석 설계 [JPA 실전(1)]


### 도메인 분석

우선 회원, 상품, 주문에 대한 도메인이 각각 필요하다. 

**회원**

- 회원 가입
- 회원 조회 (를 통해 목록을 출력한다)

상품 

- 상품 등록 C
- 상품 조회 R
- 상품 수정 U
- 상품 삭제 D

주문

- 상품 주문
- 주문 내역 조회
- 주문 취소

### 도메인 모델
![domain.png](..%2F..%2F..%2FMarkdown%2F%EB%8F%84%EB%A9%94%EC%9D%B8%20%EB%B6%84%EC%84%9D%20%EC%84%A4%EA%B3%84%20%5BJPA%20%EC%8B%A4%EC%A0%84%281%29%5D%2F%EB%8F%84%EB%A9%94%EC%9D%B8%20%EB%B6%84%EC%84%9D%20%EC%84%A4%EA%B3%84%20%5BJPA%20%EC%8B%A4%EC%A0%84%281%29%5D%20f5c1d30192564c4686a7bf837a1c2225%2Fdomain.png)

- 회원과 주문은 일 대 다 관계이다. (한 회원이 여러 개의 주문을 할 수 있기 때문에)

> 주의할 점은 ‘회원을 통해서 주문이 일어난다’가 아닌 ‘주문 할 때 회원 정보가 필요하다’ 라고 보는 것이 맞다. 인간의 관점이 아닌 시스템의 관점에서 회원과 주문은 동급이다.
> 
- 주문과 배송은 일 대 일 관계이다.
- 주문과 상품은 다 대 다 관계이다. 여러 주문이 있을 수 있으며 이 주문에 여러 개의 상품을 포함할 수 있기 때문이다.
    - 하지만 한 컬럼에는 하나의 값 밖에 적지 못한다.
    - 예를 들어, 주문에 주문하는 상품에 대한 정보를 기록한다고 해보자. 이를 1, 2, 3 으로 표현할 수 없다.
    - 따라서 주문 상품이라는 중간 테이블을 만들어서 이를 관리한다.
- 상품과 카테고리는 다 대 다 관계이다.
    - 이 또한 중간 테이블을 이용해 풀어나가야 한다.

### 엔티티 클래스 개발

이에 앞서 코드에 나왔던 주석들을 미리 예습해보자.

`@Entity` : JPA에게 해당 클래스를 엔티티로서 인식하도록 지시한다. 즉, 클래스가 데이터베이스의 테이블과 매핑되는 객체임을 알리는 것이다. 이를 통해 JPA가 테이블을 자동으로 생성하거나 혹은 이미 존재하는 테이블과 매핑한다. 

하지만 @Entity가 붙어 있다고 자동으로 테이블을 생성해주는 것은 아니다. ‘spring-boot-starter-data-jpa’와 같은 스프링 부트의 JPA 스타터 의존성을 추가해야 자동으로 데이터베이스 스키마 생성 및 업데이트 기능을 활성화 할 수 있다. 

application.yml에 작성했던 예시이다. 

```java
spring:
	jpa:  
		hibernate:    
			ddl-auto: create // update, validate, create-drop
```

`@Getter`와 `@Setter`: Lombok에서 제공하는 애노테이션으로 자동으로 Getter, Setter 메소드를 만들어준다. 프로젝트 환경에서는 사용자가 임의로 값을 세팅할 수 없게 Setter 메소드는 작성하지 않는 것이 좋다. 

`@Id` 와 `@GeneratedValue`

먼저 `@Id`는 주요 키(Primary Key)로 사용될 필드를 지정하는데 사용된다. 

❗️**여기서 잠깐**

Primary Key와 Foreign Key의 정의를 다시 잡고 가보자 

- Primary Key (PK):
    - 각 레코드를 고유하게 식별하여 테이블 내에서 중복을 방지합니다.
    - 데이터의 무결성(integrity)을 유지하기 위해 사용됩니다.
    - 데이터베이스의 인덱스를 지원하여 효율적인 데이터 검색을 가능하게 합니다.
    - 다른 테이블과의 관계를 맺을 때 FK로 사용될 수 있습니다.
- Foreign Key (FK):
    - 다른 테이블의 Primary Key 값을 참조하여 두 테이블 사이의 관계를 정의합니다.
    - 다른 테이블과의 관계를 형성하고 데이터 간의 무결성을 유지하는 데 사용됩니다.
    - 부모 테이블의 변경이나 삭제 시에 참조 무결성 제약을 통해 관련 데이터의 일관성을 보장합니다.
    - 관계형 데이터베이스에서 테이블 간의 조인(Join) 연산을 수행할 때 사용됩니다.

주의 할 점은 둘의 독립적인 관계가 아니라는 것이다. 즉, PK가 FK가 될 수 있다. 

**example)**

주문과 상품의 테이블이 있다고 가정해보자. 둘은 다대다 관계로서 중간 테이블이 필요하다. 중간 테이블을 주문 상품이라고 하자. 주문 상품은 주문 테이블의 PK인 주문_ID와 상품 테이블의 PK인 상품_ID를 FK로 참조한다. FK를 설정함으로써 두 테이블 사이의 관계를 정의한다.
이의 예제 코드를 보면 다음과 같다. 

**example 1)** 

```java
public class Member {
        @Id @GeneratedValue
        @Column(name = "member_id")
        private Long id;
}
```

**example 2)**

```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {
    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;
```

**example 3)**

```java
@ManyToOne(fetch = FetchType.LAZY) 
@JoinColumn(name = "member_id") 
private Member member; //주문 회원
```

`@GeneratedValue`: 자동으로 생성되는 ID 값을 지정한다.

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // ...
}
```

다음의 코드가 있다고 가정했을 때, ID는 1, 2, 3과 같이 순차적으로 증가하는 방식으로 할당된다. 

`@Column (name =”~”)` : name 속성을 활용하여 해당 필드와 매핑되는 데이터베이스 컬럼의 이름을 지정한다. JPA는 테이블을 생성할 때 필드의 이름을 기반으로 만들기 때문에 지정하지 않는다면 필드의 이름이 컬럼의 이름이 된다. 

name 이외에도 다양한 속성들이 있으며 이는 자주 사용되는 속성들이다. 

1. **`@Column(nullable = true)`**: **`nullable`** 속성을 사용하여 컬럼의 NULL 허용 여부를 지정합니다. **`true`**로 설정하면 NULL 값이 허용되며, **`false`**로 설정하면 NULL 값이 허용되지 않습니다.
2. **`@Column(unique = true)`**: **`unique`** 속성을 사용하여 컬럼의 고유성 여부를 지정합니다. **`true`**로 설정하면 해당 컬럼에 고유한 값만 허용되며, **`false`**로 설정하면 중복된 값이 허용됩니다.
3. **`@Column(length = 50)`**: **`length`** 속성을 사용하여 문자열 컬럼의 최대 길이를 지정합니다. 이 속성은 **`varchar`**와 같은 문자열 데이터 타입에만 적용됩니다.
4. **`@Column(columnDefinition = "TEXT")`**: **`columnDefinition`** 속성을 사용하여 컬럼의 정의를 직접 지정합니다. 이 속성을 사용하여 데이터베이스 특정 데이터 타입, 길이, 제약 조건 등을 지정할 수 있습니다.
5. **`@Column(insertable = false, updatable = false)`**: **`insertable`**과 **`updatable`** 속성을 사용하여 컬럼의 삽입 및 업데이트 가능 여부를 제어합니다. **`false`**로 설정하면 해당 컬럼은 삽입 및 업데이트 작업에서 제외됩니다.

`@Embedded` : 엔티티 클래스에 다른 객체를 내장하는데 사용된다. 예를 들어, Member 클래스에 Address 객체를 내장한다고 가정해보자.

첫 번째로 Address 클래스에 `@Embeddable` 애노테이션을 추가해야 한다.

```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
    
    // Getter, Setter, Constructors, etc.
}
```

두 번째로 이를 `@Embedded` 애노테이션을 통해 표시한다.

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @Embedded
    private Address address;
    
    // Getter, Setter, Constructors, etc.
}
```

`@OneToMany(mappedBy = “~”)`

우선 이와 비슷한 모든 애노테이션에 대해 다시 짚고 넘어가보자.

1. **`@ManyToOne`**: 일대다(N:1) 관계에서 다(N) 쪽 엔티티에서 사용되는 주석입니다. 이 주석을 사용하면 다른 엔티티(일(1) 쪽)에 대한 다(N) 쪽 엔티티의 관계를 매핑할 수 있습니다. **`@ManyToOne`** 애노테이션은 다(N) 쪽 엔티티의 필드에 지정되며, 해당 필드는 관계의 주인(Owner)이 됩니다. 대부분의 경우 **`@ManyToOne`** 애노테이션과 함께 **`@JoinColumn`** 애노테이션을 사용하여 외래 키(Foreign Key)를 지정합니다.
2. **`@OneToMany`**: 일대다(N:1) 관계에서 일(1) 쪽 엔티티에서 사용되는 주석입니다. 이 주석을 사용하면 일(1) 쪽 엔티티에서 다(N) 쪽 엔티티의 관계를 매핑할 수 있습니다. **`@OneToMany`** 애노테이션은 일(1) 쪽 엔티티의 필드에 지정되며, 해당 필드는 다(N) 쪽 엔티티와의 관계를 나타냅니다. 대부분의 경우 **`mappedBy`** 속성을 사용하여 양방향 관계를 설정하고, **`@JoinColumn`** 애노테이션을 사용하여 외래 키를 지정합니다.
3. **`@OneToOne`**: 일대일(1:1) 관계에서 사용되는 주석입니다. 이 주석을 사용하면 두 엔티티 간의 일대일 관계를 매핑할 수 있습니다. **`@OneToOne`** 애노테이션은 각 엔티티의 필드에 지정되며, 관계의 주인(Owner)을 명시적으로 지정할 수 있습니다. 일대일 관계에서는 **`@JoinColumn`** 애노테이션을 사용하여 외래 키를 지정할 수 있습니다.
4. **`@JoinColumn`**: 외래 키(Foreign Key)를 지정하기 위해 사용되는 주석입니다. 일대다(**`@OneToMany`**), 다대일(**`@ManyToOne`**), 일대일(**`@OneToOne`**) 관계에서 주로 사용됩니다. **`@JoinColumn`** 애노테이션은 관계의 주인(Owner) 엔티티의 필드에 지정되며, 외래 키를 매핑할 컬럼의 이름, 외래 키의 속성 등을 지정할 수 있습니다.

다시 돌아와서 Member ↔ Order 관계를 살펴보자.
Member가 1, Orders가 다 관계이다. 

Member 입장에서 Order는 `@OneToMany` 일 것이고, Order 입장에서 Member는 `@ManyToOne` 일 것이다. 

따라서 Member는 한 회원이 여러 주문을 할 수 있기에 다음과 같이 리스트의 형태로 받고

```java
@OneToMany(mappedBy = "member")
private List<Order> orders = new ArrayList<>(); 
```

Order는 JoinColumn을 통해 member_id를 FK로 지정한다.

```java
@ManyToOne(fetch = FetchType.LAZY) 
@JoinColumn(name = "member_id") 
private Member member; //주문 회원
```

> Member의 mappedBy = “member”는 Order의 private Member member 필드를 기반으로 연관 관계를 맺을 것이다.
> 

`cascade = CascadeType.ALL 속성` :

**`cascade = CascadeType.ALL`**은 JPA에서 연관된 엔티티들 간의 연산을 일괄적으로 처리하기 위해 사용되는 속성이다. 이 속성을 사용하면 한 엔티티의 변경이나 삭제 작업이 관련된 다른 엔티티에도 전파되어 해당 연관 엔티티들의 상태를 자동으로 변경할 수 있다.

Q. 연관 관계 메서드는 어디에 쓰이는가?

A. 주로 연관 관계의 주인 엔티티에서 정의된다. 쉽게 생각하면 일대다의 관계에서는 set, 다대일의 관계에서는 add를 쓴다. 예제로서 확인해보자

Orders는 다음과 같은 연관 관계 메서드들을 정의하고 있다.

```java
public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
}

public void addOrderItem(OrderItem orderItem) {
				 orderItems.add(orderItem);
         orderItem.setOrder(this);
}

public void setDelivery(Delivery delivery) {
          this.delivery = delivery;
          delivery.setOrder(this);
}
```

setMember는 현재 주문이 어떤 멤버에게 속하는지 설정하는 역할을 한다.

addOrderItem은 현재 주문에 상품 주문을 추가하는 역할을 한다. 


Q. 연관 관계 메서드는 한 엔티티 클래스에 몰아서 작성하는게 좋나?

A. 일반적으로는 한 엔티티 클래스에 연관 관계 메서드를 몰아서 작성하는 방식이 가독성과 일관성을 유지하기 위해 권장된다. 즉, 해당 엔티티 클래스 내에서 연관 관계를 설정하고 변경하는 메서드들을 함께 위치시킴으로써 관련 코드를 한 곳에서 찾을 수 있다. 이는 개발자의 선호도에 따라 다르다.