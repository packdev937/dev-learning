```
> No tests found for given includes: [gundorit.batch.BatchJobTest](--tests filter)
```

다음과 같은 예외가 발생했을 때 

```java
import org.junit.Test;
```

위를 다음과 같이 고쳐줍니다.
```java
import org.junit.jupiter.api.Test;
```
