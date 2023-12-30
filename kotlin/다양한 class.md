### Enum Class
Enum 클래스의 간단한 예제입니다.
```kotlin
enum class Color {
	RED,
	GREEN,
	BLUE
}
```

이는 다음과 같이 확장할 수 있습니다.
```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```


### Data Class
자바의 Record와 비슷하며 몇 가지 차이점을 가지고 있습니다.
-  kotlin의 `data` 클래스는 다른 클래스를 상속할 수 없지만, 다른 클래스를 상속하거나 인터페이스를 구현할 수 있습니다. 반면, Java의 `record`는 다른 클래스를 상속할 수 없으며, 오직 인터페이스만 구현할 수 있습니다.
- kotlin의 `data` 클래스에서는 `equals()`, `hashCode()`, `toString()` 메서드를 재정의할 수 있습니다. 하지만 Java의 `record`에서는 이러한 메서드들을 재정의할 필요가 없으며, 사실상 할 수도 없습니다.
- kotlin의 `data` 클래스에서는 주 생성자에서 복잡한 로직을 수행할 수 있습니다. Java의 `record`에서는 주 생성자의 로직이 매우 제한적입니다.

예제 코드로 살펴보겠습니다.
```kotlin
data class User(val name: String, val age: Int)
```

이는 다음과 같이 호출 가능합니다
```kotlin
fun main() {
    // 인스턴스 생성
    val user1 = User("Alice", 30)
    val user2 = User("Bob", 25)

    // toString() 메서드를 사용한 출력
    println(user1)  // 출력: User(name=Alice, age=30)
    println(user2)  // 출력: User(name=Bob, age=25)

    // equals() 메서드를 사용한 비교
    val user3 = User("Alice", 30)
    println(user1 == user3)  // 출력: true

    // copy() 메서드 사용
    val user4 = user1.copy(name = "Charlie")
    println(user4)  // 출력: User(name=Charlie, age=30)
}

```