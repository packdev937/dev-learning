### function
function의 기본 형태는 다음과 같습니다.
```kotlin
fun 함수명 (매개변수: 타입): 반환타입 {
	return 반환값
}
```

예제 코드로 알아봅시다.
```kotlin
fun main(args: Array<String>) {
	val price1: Int = 10000
	val price2: Int = 20000

	val price3 = sumPrice(price1, price2)
}

fun sumPrice(price1: Int, price2: Int): Int {
	return price1 + price2
}	
```


위 sumPrice 함수는 다음과 같이 변경될 수 있습니다.
```kotlin
fun sumPrice(price1: Int, price2: Int): Int = price1 + price2
```

이 때, price1 과 price2 모두 Int 타입이기 때문에 반환 타입 또한 생략 가능합니다.
``` kotlin
fun sumPrice(price1: Int, price2: Int) = price1 + price2
```

### class
기본 형태는 다음과 같습니다.
참고로 클래스의 프로퍼티는 선언과 동시에 초기화 해주어야 합니다.
```kotlin
class 클래스명 {
}
```

이를 응용해서 다음 Person 클래스를 만들 수 있습니다.
```kotlin
class Person {

	constructor (name: String, age: Int) {
	    this.name = name
	    this.age = age
	  }
	  
	  var name: String = ""
	  var age: Int = 0
}
```

생성자는 클래스 이름 옆에 선언할 수 있습니다.
```kotlin
class Person (name: String, age: Int) {
  var name: String = name
  var age: Int = age
}
```

인스턴스화는 다음과 같이 할 수 있습니다.
```kotlin
fun main(args: Array<String>) {
	val person = Person("홍길동", 25)
}
```

### Interface
인터페이스의 기본 형태는 다음과 같습니다.
```kotlin
interface 인터페이스명 {
	fun 함수명()
}
```

인터페이스의 꽃은 상속입니다. 이를 코드로서 알아보겠습니다.
```kotlin
interface Item {
	fun buy()
	fun sell()
}

class Phone(val name: String, val price: Int): Item {
	override fun buy(){
		println("buy $name")
	}

	override fun sell(){
		println("sell $price dollar")
	}
}
```