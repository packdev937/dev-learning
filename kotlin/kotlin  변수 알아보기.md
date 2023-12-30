### var와 val 
> 변수를 선언할 때 사용하는 키워드

변수 선언의 기본 형태는 다음과 같습니다. 
```kotlin
var 변수명: 타입
```

`var`는 읽기 쓰기가 가능한 변수를 선언할 때 사용합니다.
```kotlin
var a = 1
a = 2
```

반면 `val`은 읽기만 가능한 변수로 초기화 후 재 할당이 불가합니다. 단, 초기화 시 최초 한번 할당이 가능합니다.
```kotlin
val a = 1
a = 2 // Error
```

만약 후에 초기화를 진행한다고 가정하면 타입을 명시해주어야 합니다.
```
val a : Int
a = 1
```

*그렇다면 val은 java로 치면 상수의 역할을 하고 있는건가?*
그렇습니다. `val`은 java의 `final`과 유사한 역할을 합니다.

### ? 
kotlin의 Int는 null을 허용하지 않습니다. 
null을 허용하기 위해서는 타입에 ? 를 붙여서 선언해야 합니다. 
```kotlin
var i: Int? = 10
var j: Int = 20

i = null
j = null // Error 
```

이는 String도 마찬가지입니다. 
```kotlin
var i: String? = "Hello"
var j: String = "World!"

i = null
j = null // Error
```

### ?/ ?. / ?: / !!
?에서 더 나아가 ?., ?:, !! 에 대해 알아보도록 하겠습니다.

?. 은 null safety operator 입니다. 

예를 들어 보겠습니다.
```kotlin
fun getLen(str: String?) = str.length
```

위와 같은 함수가 있을 때 str에 null이 들어오면 NPE 이 발생합니다. 

```kotlin
fun getLen(str: String?) = str?.length
```

?: 는 elvis operator 입니다. 

위 연산자는 null 값이라면 default로 다른 값을 주는 것을 의미합니다.
``` kotlin
 fun getName(str: String?) {
    val name = str ?: "Unknown"
}
```

!! 은 강제로 null이 아님을 선언합니다. 

더 자세한 내용은 다음을 확인해보면 좋을 것 같습니다
- https://tourspace.tistory.com/114
### 타입 추론
kotlin은 기본적으로 타입 추론이 가능합니다. 타입 추론이란 타입을 명시하지 않아도 kotlin이 이를 자동으로 추론하는 것을 의미합니다. 
```kotlin
val s = "Hello"
val i = 1
val l = 1L
val d = 1.0
val f = 1.0f

println(s::class)
// 이하 생략 
```

### print()
kotlin의 print()는 다음과 같이 사용할 수 있습니다. 
```kotlin
var s: String = "World"
var w: String = "!"
println("Hello " + s)
println("Hello $s")
println("Hello ${s+w}")
println("Hello ${s+w}"+ "Bye")
```

### in
```kotlin
val price: Int = 100

if(price in arrayOf(100, 200, 300)) {
	println("Array contains $price")
} else {
	println("NOT CONTAINED")
}
```

### when 
```kotlin
val price: Int = 100

when(price){
	100 -> println("price is 100")
	200 -> println("price is 200")
	300 -> println("price is 300")
	else -> println("NONE")
}
```

*kotlin의 when은 switch랑 같은 역할을 하는가?*
유사하지만 kotlin이 조금 더 유연하고 강력한 기능을 제공합니다.

- `when`은 단순한 값 뿐만 아니라 범위(range), 타입 체크, 심지어 임의의 표현식도 조건으로 사용할 수 있습니다.
- `when`은 값을 반환하는 표현식으로 사용될 수 있으며, 이를 통해 코드를 더 간결하게 작성할 수 있습니다.
- 코드 블럭을 지원하며 각 분기에서 단일 표현식 뿐만 아니라 복수의 명령문도 사용할 수 있습니다.

예를 들어, Kotlin에서 `when`을 사용하는 방법은 다음과 같습니다:

```kotlin
val x = 5

when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    in 3..10 -> print("x is in the range 3 to 10")
    is String -> print("x is a String")
    else -> { 
        print("x is neither 1 nor 2 nor in the range")
    }
}
```
