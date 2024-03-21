간단한 자바 프로젝트를 코틀린으로 리팩토링 하는 과정을 담아보겠습니다.
### Domain
	POJO, JPA Entity Object
##### 1. Java Entity를 Kotlin Entity로 
먼저 Book 이라는 도메인을 리팩토링 해보도록 하겠습니다.
아래와 같이 코드를 구상하면 다음과 같은 에러를 발생합니다. 
```kotlin
@Entity  
class Book(  
    val name: String,  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    val id: Long? = null,  
) {  
}
```
![[kopring-refactoring exception.png]]

이는 build.gradle에 다음 코드를 추가해주면 됩니다.
```kotlin
plugins {  
    id 'org.springframework.boot' version '2.6.8'  
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'  
    id 'java'  
    id 'org.jetbrains.kotlin.jvm' version '1.6.21'  
    // 추가된 부분 
    id 'org.jetbrains.kotlin.plugin.jpa' version '1.6.21'  
}
```

+추가 
아래 의존성을 추가하지 않는다면 @Entity를 인식하지 못하는 것 같습니다. 
```kotlin
implementation 'org.jetbrains.kotlin:kotlin-reflect:1.2.41'
```
최종 리팩토링 코드는 다음과 같습니다.
```kotlin
@Entity  
class Book(  
    val name: String,  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    val id: Long? = null,  
) {  
    init {  
        if (name.isBlank()) {  
            throw IllegalArgumentException("이름은 비어 있을 수 없습니다")  
        }  
    }  
}
```

자바에 비해서 많이 간결해진 모습입니다. 
```java
@Entity  
public class JavaBook {  
  
  @Id  
  @GeneratedValue(strategy = IDENTITY)  
  private Long id;  
  
  @Column(nullable = false)  
  private String name;  
  
  public JavaBook() {  
  
  }  
  
  public JavaBook(String name) {  
    if (name.isBlank()) {  
      throw new IllegalArgumentException("이름은 비어 있을 수 없습니다");  
    }  
    this.name = name;  
  }  
  
  public String getName() {  
    return name;  
  }  
  
}
```

이제 Repository > Service > Controller 순으로 리팩토링 된 Book 객체를 넣어주어야 합니다.
### Repository
	Spring Bean, 의존 독립성
Java Repository를 Kotlin Repository로 바꾸는 방법은 간단합니다.
예를 들어, 다음 UserRepository를 Kotlin으로 리팩토링 한다고 가정해보겠습니다.
```java
public interface UserRepository extends JpaRepository<User, Long> {

	public Optional<User> findByName(String name);
}
```

이를 다음과 같이 나타낼 수 있습니다.
```kotlin
interface UserRepository : JpaRepository<User, Long> {  
  
    fun findByName(name: String): Optional<User>  
}
```

`Optional<User>` 또한 코틀린 문법의 null 연산자를 사용해서 대체할 수 있습니다. 
### Service
	Spring Bean, 비즈니스 로직, 다른 Bean들을 의존
##### 1. Default Parameter
```kotlin
class Book(  
    val name: String,  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    val id: Long? = null,  
) 
```

여기서 Long은 default parameter인데요, 자바 -> 코틀린으로 변경하는 과정에서 이를 인식하지 못합니다. 따라서 다음과 같은 default parameter는 직접 명시해주어야 합니다.
```java
@Transactional  
public void saveBook(BookRequest request) {  
  Book newBook = new Book(request.getName(), null);  
  bookRepository.save(newBook);  
}
```
### Controller
	Spring Bean, 의존, 많은 DTO 클래스

#### 추가 
자바의 람다를 리팩토링
```java
public void returnBook(String bookName) {  
  UserLoanHistory targetHistory = this.userLoanHistories.stream()  
      .filter(history -> history.getBookName().equals(bookName))  
      .findFirst()  
      .orElseThrow();  
  targetHistory.doReturn();  
}
```
다음 코드는 이렇게 리팩토링 가능합니다. 
```kotlin
fun returnBook(bookName: String){  
    this.userLoanHistories.first{  
        history -> history.bookName == bookName  
    }.doReturn()  
}
```

null을 담지 못할 때
UserUpdateRequest를 사용해야 하는데 UserUpdateRequest의 필드가 long 이었다습니다. 중요한건 long은 null을 담을 수 없었습니다.

하지만 코드가 실행되고 확정적으로 null이 아닌 값이 들어갈 때 임시로 !!를 붙여줄 수 있습니다.
```kotlin
fun 유저_이름을_업데이트한다() {  
    // given  
    val savedUser = userRepository.save(User("A", 20))  
    val request = UserUpdateRequest(savedUser.id!!, "B")

	...
```