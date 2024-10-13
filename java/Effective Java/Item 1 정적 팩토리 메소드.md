## 1. 개요
제 1 아이템. 생성자 대신 정적 팩토리 메소드를 생성해야 하는 이유를 깊게 알아보고자 합니다. 

## 2. 정적 팩토리 메소드는 이름을 가질 수 있습니다. 

일반적으로 클래스를 정의하고 생성자를 만들면 다음과 같습니다. 

```java
class Student {
	private String name;
	private String major;
	private String year;

	public Student(String name, String major, String year) {
		this.name = name;
		this.major = major;
		this.year = year;
	}
}
```

위처럼 기본 생성자를 사용하면, **생성자가 여러 개 있을 때** 그 의도를 파악하기 어려울 수 있습니다. 현재는 생성자의 매개변수가 단 3개 뿐이지만, 생성자의 매개변수가 많아질 때 어떤 생성자가 어떤 목적으로 사용되는지 알기 힘듭니다.

```java
class Student {
    private String name;
    private String major;
    private String year;  // 학년 (Freshman, Sophomore, Junior, Senior)

    // private 생성자
    private Student(String name, String major, String year) {
        this.name = name;
        this.major = major;
        this.year = year;
    }

    // 정적 팩토리 메소드
    public static Student createFreshman(String name, String major) {
        return new Student(name, major, "Freshman");
    }

    public static Student createSophomore(String name, String major) {
        return new Student(name, major, "Sophomore");
    }

    public static Student createJunior(String name, String major) {
        return new Student(name, major, "Junior");
    }

    public static Student createSenior(String name, String major) {
        return new Student(name, major, "Senior");
    }
}
```

정적 팩토리 메소드를 사용하면 보다 더 직관적인 객체를 생성할 수 있습니다. 즉, 객체의 특성을 더 설명할 수 있습니다. 

## 3. 호출 될 때 마다 인스턴스를 새로 생성하지 않는다

정적 팩토리 메소드를 사용하면 **새로운 인스턴스를 매번 생성하지 않고** **이미 생성된 인스턴스를 반환**할 수 있습니다. 이것이 어떤 의미를 가질까요?

정적 팩토리 메소드는 **내부적으로 객체를 생성한 후 이를 캐싱**할 수 있습니다. 다음 호출 시에는 **이미 캐싱된 객체를 반환**하는 방식으로 동작할 수 있습니다. 이를 통해 **매번 새로운 인스턴스를 생성할 필요가 없어지며**, 불필요한 메모리 할당과 CPU 연산이 줄어듭니다.

대표적인 예시 중 하나인 `Boolean.valueOf` 입니다.
```java
public final class Boolean {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return b ? TRUE : FALSE;  // 기존 객체를 반환
    }
}
```

생성 비용이 큰 객체가 자주 참조된다면 위와 같은 캐싱을 시도해봅시다.

생성 비용이 큰 객체의 예시는 다음과 같습니다. 
1. 데이터베이스 연결 객체
2. 파일 시스템 관련 객체
3. 복잡한 초기화가 필요한 객체
4. API 호출 객체
5. 대규모 연산을 수행하는 객체 
6. 정규 표현식 객체 

특히 다양하게 사용될 수 있는 `Pattern`의 예제는 다음과 같습니다. 
```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexCache {

    // 패턴을 캐싱하는 Map
    private static final Map<String, Pattern> patternCache = new ConcurrentHashMap<>();

    // 캐시에서 패턴을 가져오거나, 없다면 컴파일 후 캐싱
    public static Pattern getPattern(String regex) {
        return patternCache.computeIfAbsent(regex, Pattern::compile);
    }

    public static Matcher getMatcher(String regex, String input) {
        Pattern pattern = getPattern(regex);
        return pattern.matcher(input);
    }
}

public class Main {
    public static void main(String[] args) {
        String regex = "[a-z]+";
        String input = "example";

        // 패턴 캐싱 및 재사용
        Matcher matcher = RegexCache.getMatcher(regex, input);
        
        if (matcher.matches()) {
            System.out.println("매칭됨!");
        } else {
            System.out.println("매칭되지 않음.");
        }

        // 동일한 정규 표현식 재사용 (캐싱됨)
        Matcher matcher2 = RegexCache.getMatcher(regex, "anotherexample");
        if (matcher2.matches()) {
            System.out.println("또다시 매칭됨!");
        }
    }
}

```

+참고로 스프링 부트 환경에서 @Bean으로 등록한 싱글톤 객체도 이러한 성능 최적화의 이점을 얻을 수 있습니다.

## 4. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. 

정적 팩토리 메소드는 **반환 타입을 더 유연하게 설계**할 수 있는 장점을 가지고 있습니다. 즉, **부모 클래스나 인터페이스**를 반환 타입으로 선언한 후, **구체적인 하위 클래스 객체**를 반환할 수 있다는 의미입니다.

일반적으로 **생성자**는 명확하게 해당 클래스의 인스턴스만 반환할 수 있습니다. 하지만 **정적 팩토리 메소드**는 **인터페이스**나 **상위 클래스**를 반환 타입으로 설정해두고, **실제 구현체**로 **하위 타입**의 객체를 반환할 수 있습니다. 이는 **캡슐화**를 강화하고, **구현 세부 사항**을 숨기며, **유연한 객체 생성**을 가능하게 합니다.

일반 생성자를 사용하게 되면 `Animal`의 생성자는 **항상 Animal 객체**만 반환합니다. 예를 들어, `new Animal("Dog")`는 **항상 Animal 인스턴스**를 반환하게 됩니다.
```java
class Animal {
    private String name;
    public Animal(String name) {
        this.name = name;
    }
}

class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
}

```

하지만 정적 팩토리 메소드를 사용하게 되면 동적인 객체 생성이 가능합니다.
```java
interface Animal {
    void sound();
}

class Dog implements Animal {
    @Override
    public void sound() {
        System.out.println("Bark");
    }
}

class Cat implements Animal {
    @Override
    public void sound() {
        System.out.println("Meow");
    }
}

class AnimalFactory {

    public static Animal createAnimal(String type) {
        if (type.equalsIgnoreCase("dog")) {
            return new Dog();  // Dog 객체 반환 (Animal 하위 타입)
        } else if (type.equalsIgnoreCase("cat")) {
            return new Cat();  // Cat 객체 반환 (Animal 하위 타입)
        } else {
            throw new IllegalArgumentException("Unknown animal type");
        }
    }
}
```

또한 나중에 **Animal의 하위 타입**을 추가하거나 변경할 때, 기존 코드 수정 없이 **정적 팩토리 메소드**의 내부 구현만 수정하면 됩니다.

대표적인 예시는 JDBC의 DriverManager 입니다.

JDBC의 `DriverManager.getConnection()` 메소드는 대표적인 **정적 팩토리 메소드**의 예시입니다. 이 메소드는 **Connection 인터페이스**를 반환하지만, 실제로는 **다양한 하위 타입의 Connection 객체**를 반환할 수 있습니다. 이 방식으로 **JDBC 드라이버에 따라 다른 Connection 구현체**를 반환하면서도 호출자는 구체적인 구현을 신경 쓸 필요가 없게 됩니다.

```java
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "user", "password");
// conn은 Connection 인터페이스를 통해 동작하지만, 실제 구현체는 MySQLConnection일 수 있습니다.

```

또한, 자바 8 이후로 인터페이스를 통해 정적 메소드를 호출할 수 있게 되었습니다. 

기존에는 인스턴스화가 불가능한 (private 생성자를 곁들인) 다음과 같은 클래스를 구현해야 했다면
```java
public class MathUtils {

    // private 생성자: 인스턴스화 불가
    private MathUtils() {
        throw new AssertionError("Cannot instantiate this class");
    }

    // 정적 메소드
    public static int add(int a, int b) {
        return a + b;
    }

    public static int subtract(int a, int b) {
        return a - b;
    }
}
```

이제는 인터페이스에 정적 메소드를 선언하는 것으로 해결할 수 있습니다.
```java
public interface Calculator {

    // 정적 메소드
    static int add(int a, int b) {
        return a + b;
    }

    static int subtract(int a, int b) {
        return a - b;
    }
}
```

## 5. 정적 팩토리 메소드 관례 
`from`, `of`, `valueOf`, `instance`, `getInstance`, `create`, `getType`, `newType`, `type`