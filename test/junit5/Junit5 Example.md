주어진 값이 같은 지 확인
```java
assertThat(number).isEqualTo(8)
```

주어진 값이 true / false 인지 검증
```kotlin
assertThat(isTested).isTrue
assertThat(isTested).isFalse
```

주어진 컬렉션의 크기를 확인
```kotlin
assertThat(people).hasSize(2)
```

주어진 컬렉션 안의 item들에서 name 이라는 프로퍼티를 추출한 후, 그 값이 A 와 B 인지 검증
```kotlin
val people = ListOf(Person("A"), Person("B"))
assertThat(people).extracting("name").containsExactlyInAnyOrder("A", "B")
```

주어진 컬렉션 안의 item들에서 name 이라는 프로퍼티를 추출한 후, 그 값이 A 와 B 인지 검증 (이때 순서도 중요하다)
```kotlin
val people = ListOf(Person("A"), Person("B"))
assertThat(people).extracting("name").containsExactly("A", "B")
```

지정된 Exception이 발생하는지 확인 
```kotlin
asserThrows<IllegalArgumentException> {
	function1()
}
```

예외 메세지를 검증
```kotlin
val message = assertThrows<IllegalArgumentException> {
	function1()
}.message

assertThat(message).isEqaulTo("...")
```

