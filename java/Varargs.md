> Verargs를 한국어로 번역하면 가변 인자 입니다. Java 5 에서 도입된 기능이며, 필요에 따라 매개 변수의 개수를 가변적으로 조정할 수 있습니다.

### 예제 코드로 알아보는 Varargs
```java
void func(String... inputs){
	for(String input : inputs){
		System.out.print(input+" ");		
	}
}

func("hello"); // hello
func("hello", "world") // hello world
func("hello", "world", "!") // hello world ! 
```

여기서 알아볼 수 있는 이점은 다음과 같습니다.
*"반복적인 오버로딩을 막을 수 있다"*

예를 들어, 위 코드를 오버로딩으로 구현하려고 한다면 다음과 같이 코드를 작성해야 합니다. 

```java
void func(String string1){
    //Do Something
}
void func(String string1, String string2){
    //Do Something
}
void func(String string1, String string2, String string3){
    //Do Something
}
```

### 단점도 있는 가변인자

가변 인자로 선언된 함수를 호출하면 컴파일러가 함수 실행 직전에 배열을 생성하도록 코드를 변경합니다. 이는 성능 상의 문제가 될 수 있습니다. 

특히 반복 문 안에서 실행하는 경우에는 매 실행 마다 배열을 생성하기 때문에 성능 문제가 더 커질 수 있습니다. 

특정 메서드를 실행하는데 일정 개수 이상의 인자의 개수가 필요하지 않다면 함수 오버로딩으로 생성하는게 효과적 입니다. 

![[Java Docs - Map.of.png]]

Java 9에 추가된 `Map.of` 의 일부입니다. Map.of 함수는 인자가 0개 부터 10개 까지는 오버로딩으로 생성되어 있습니다. 

### Reference
https://velog.io/@kasania/Varargs%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B4%80%EC%B0%B0