## 1. Error와 Exception 정의 
Error는 시스템에 비정상적인 상황이 발생한 경우를 의미합니다. 주로 Java Virtual Machine에서 발생하며 어플리케이션 코드에서 잡을 수 없습니다. 대표적인 예시로는 OutOfMemoryError, StackOverFlow 등이 있습니다. 

Exception은 프로그램 흐름에 어긋나는 경우를 의미합니다. 예를 들어서, 잘못된 입력 값에 대한 IllegalArgumentException이나 null 값에 대한 NullPointerException 등이 존재합니다. 
개발자는 이러한 예외들을 미리 예측하고 직접 핸들링 할 수 있습니다. 

## 2. 예외의 종류 
예외의 종류로는 CheckedException과 UncheckedException으로 나눌 수 있습니다. 

CheckedException은 RuntimeException을 상속받지 않은 클래스이며, 컴파일 시점에 확인할 수 있으며 명시적인 예외 처리를 할 수 있습니다. 
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class CheckedExceptionExample {
    public static void main(String[] args) {
        try {
            BufferedReader reader = new BufferedReader(new FileReader("test.txt"));
            String line = reader.readLine();
            System.out.println(line);
            reader.close();
        } catch (IOException e) {
            System.err.println("An IOException was caught: " + e.getMessage());
        }
    }
}

```

UnCheckedException은 RuntimeException을 상속받은 클래스이며 런타임 싲덤에만 확인할 수 있습니다. 
```java
public class UncheckedExceptionExample {
    public static void main(String[] args) {
        String str = null;
        try {
            System.out.println(str.length());
        } catch (NullPointerException e) {
            System.err.println("A NullPointerException was caught: " + e.getMessage());
        }
    }
}

```