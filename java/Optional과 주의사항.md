Java 8에서 도입된 `Optional` 클래스는 값이 존재할 수도 있고, 존재하지 않을 수도 있는 상황에서 사용됩니다. 이는 값이 없는 상황을 명시적으로 나타내어 `NullPointerException`으로부터 안전하게 만들어줍니다. `Optional` 클래스를 사용하면 값이 없을 때의 처리를 더 간결하고 명확하게 할 수 있습니다.

1. **Null 체크의 대체**:
   기존의 null 체크를 사용하는 대신, `Optional`을 사용하여 더 읽기 쉬운 코드를 작성할 수 있습니다.
   ```java
   // 기존 방식
   if (a == null) {
       // null 처리 로직
   }

   // Optional 방식
   if (a.isEmpty()) {
       // 빈 값 처리 로직
   }
   ```

2. **값의 존재를 명시적으로 표현**:
   `Optional`을 사용하면 값을 반환하지 못하는 경우를 명시적으로 나타낼 수 있습니다. 예를 들어, 데이터베이스 조회나 외부 API 호출 결과가 없을 때 유용합니다.
   ```java
   Optional<String> optionalValue = findValueByKey("key");
   if (optionalValue.isPresent()) {
       // 값이 존재할 때 처리 로직
       String value = optionalValue.get();
   } else {
       // 값이 없을 때 처리 로직
   }
   ```

3. **기본값 설정 및 처리**:
   `Optional` 클래스는 값이 없을 경우 기본값을 설정하거나, 값이 존재할 때만 특정 로직을 수행하는 메서드를 제공합니다.
   ```java
   String value = optionalValue.orElse("기본값");
   
   optionalValue.ifPresent(val -> {
       // 값이 존재할 때 수행할 로직
       System.out.println(val);
   });
   ```

4. **함수형 스타일의 처리**:
   `Optional`은 함수형 프로그래밍 스타일로 값을 처리할 수 있는 다양한 메서드를 제공합니다.
   ```java
   optionalValue
       .map(String::toUpperCase)
       .ifPresent(System.out::println);
   ```

### 주의할 점
- 값의 존재 여부를 판단하지 않고 접근하려 한다면 NPE는 피하지만 NoSuchElementException이 발생할 수 있습니다. 
- Optional 인스턴스 자체가 null 일 수도 있습니다. 
- Optional은 객체를 감싸는 Wrapper 클래스 이기 때문에 객체를 저장하기 위한 추가 메모리가 필요하고, 또한 Optional 안에 있는 객체를 얻기 위해서 접근 해야 하므로 추가적인 접근 비용이 증가합니다. 이는 오버헤드를 야기할 수 있습니다.  