### 개요

JPA가 이미 시장에 자리를 잡고 배우는 저로서는 김영한님이 강조하시는 JPA의 소중함을 제대로 느끼지 못하고 있었습니다. 무언가의 가치를 알고 그것을 효과적으로 사용하기 위해서는 조금 무식한(?) 방법일지라도 어렵게 배우는 것이 좋다고 생각합니다. 따라서 **자바 ORM 표준 JPA 프로그래밍**에 등장하는 예제들을 시각화하여 JPA의 소중함을 조금 더 느끼고자 글을 작성하게 되었습니다.

### find

무언가를 조회하고 싶을 때 JPA를 사용하면 다음 한 줄로 끝이 납니다.

```java
@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager entityManager;

    public Member find(Long id) {
        return entityManager.find(Member.class, id);
    }
}
```

반면에 JPQL을 사용해서 조회를 하려면 어떻게 해야 할까요?

```java
public Member find(Long id) {
    String sql = "SELECT MEMBER_ID, NAME FROM Member M WHERE MEMBER_ID = ?";
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    Member member = null;

  
    pstmt = connection.prepareStatement(sql);
    pstmt.setLong(1, id);
    rs = pstmt.executeQuery();

    if (rs.next()) {
	       String memberId = rs.getString("MEMBER_ID");
	       String name = rs.getString("NAME");

	       member = new Member();
         member.setId(memberId);
         member.setName(name);
     
    return member;
}
```

예제를 작성하는 동안 벌써 JPA가 그리워졌는데요, 위의 수 많은 코드가 JPA를 사용하면 단축되는 것을 확인할 수 있습니다.

진짜 문제는 다음 경우에 발생합니다.

“Member에 전화번호도 추가해주세요!”

이는 단순히 Member에 phone 필드만 추가하는 것이 아닌 SQL문을 전반적으로 수정해야 합니다. 김영한님은 이를 **SQL에 의존적인 개발**이라고 언급하셨는데 직접 코드를 쳐보니 확연히 알 수 있는 대목 같습니다.

다시 **JPA**로 돌아와서 **JPA**는 어떻게 이를 가능하게 할까요? 우선 JPA은 개발자가 직접 SQL문을 작성하는 것이 아닌 API가 적절한 **SQL**을 생성해줍니다.

