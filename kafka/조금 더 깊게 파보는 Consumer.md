## 1. 개요
아래 그림은 Consumer와 Consumer Group을 시각화 한 것입니다. 
![[Pasted image 20240906202742.png|500]]
## 2. Spring Boot에서 Consumer 설정은 어떻게 할까?
```java
@Bean  
public ConsumerFactory<String, Object> consumerFactory() {  
    Map<String, Object> config = new HashMap<>();  
  
    config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");  
    config.put(ConsumerConfig.GROUP_ID_CONFIG, "feed-service");  
    config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringSerializer.class);  
    config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonSerializer.class);  
    config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
    config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
  
    return new DefaultKafkaConsumerFactory<>(config);  
}

@Bean  
public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory(
	ConsumerFactory<String, Object> consumerFactory
) {  
    ConcurrentKafkaListenerContainerFactory<String, Object> factory =
     new ConcurrentKafkaListenerContainerFactory<>();  
    
    factory.setConsumerFactory(consumerFactory); // 컨슈머 팩토리 설정
    factory.setConcurrency(3); // 동시에 3개의 스레드를 사용하여 메시지를 처리하도록 설정
    factory.getContainerProperties().setPollTimeout(3000); // 메시지를 폴링 시간 설정
  
    return factory;  
}
```

위의 설정이 Consumer의 모든 설정을 커버하는 건 아니지만 우선 차근차근 알아보도록 하겠습니다. 

### 2-1. `ConsumerFactory<String, Object>` 
`ConsumerFactory<String, Object>` 는 Kafka 컨슈머를 생성하는 팩토리 클래스입니다. 컨슈머가 Kafka 브로커로 부터 메세지를 가져오기 위한 설정을 관리하고 컨슈머 인스턴스를 생성합니다. 해당 팩토리를 통해 생성된 컨슈머는 Kafka 토픽에서 메세지를 읽고 어플리케이션을 처리합니다. 

사용되는 간단한 예제는 다음과 같습니다. 
```java
@KafkaListener(topics = "created_photo_topic", 
			   groupId = "feed-service", 
			   containerFactory = "kafkaListenerContainerFactory"
			   ) 
public void listen(String message) { 
	System.out.println("Received message: " + message);
}
```

`kafkaListenerContainerFactory` 이거는 위에 정의된 Bean 입니다. ConsumerFactory 인스턴스를 생성하고 이를 `.setConsumerFactory()`를 통해 매핑하는 코드를 볼 수 있습니다. 

그렇게 생성된 `kafkaListenerContainerFactory`은 `@KafkaListner`의 파라미터로 사용됩니다. 

즉, 리스너가 사용할 Kafka 컨슈마 설정을 명시한다는 것인데, 이를 다르게 말하면 각 리스너에 다른 컨슈머 설정을 매핑할 수 있습니다. 

### 2-2. `ConcurrentKafkaListenerContainerFactory<String, Object>`
`ConcurrentKafkaListenerContainerFactory`는 KafkaListener가 사용할 컨슈머의 설정을 관리하고, 메시지를 병렬로 처리할 수 있도록 해주는 팩토리입니다. Kafka에서 메시지를 처리할 때 사용될 리스너의 동작 방식을 설정하는 데 매우 중요한 역할을 합니다.

**🤔 리스너와 컨슈머는 다른가**
개념적으로 다르지만 둘은 밀접한 연관이 있습니다. 

**컨슈머**는 Kafka 브로커로 부터 메세지를 읽어오는 주체입니다. 특정 토픽에서 메세지를 소비하는 역할을 합니다. Kafka API를 통해 직접 메세지를 풀링할 수도 있습니다. 
```java
Consumer<String, String> consumer = new KafkaConsumer<>(config);
consumer.subscribe(Arrays.asList("my-topic"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println("Consumed message: " + record.value());
    }
}
```

리스너는 컨슈머의 상위 개념으로, Kafka에서 메세지를 수신하는 작업을 자동화 하고 편리하고 관리하는 역할을 합니다. 리스너는 컨슈머의 폴링 작업을 자동으로 처리하고, 메시지가 들어오면 이를 콜백 메서드를 통해 자동으로 처리해 줍니다. 스프링 Kafka에서는 `@KafkaListener`를 사용하여 리스너를 정의할 수 있습니다.

```java
@KafkaListener(topics = "created_photo_topic", 
			   groupId = "feed-service", 
			   containerFactory = "kafkaListenerContainerFactory"
			   ) 
public void listen(String message) {
    System.out.println("Received message: " + message);
}
```

즉, 컨슈머는 리스너의 내부에서 동작합니다. 리스너는 메시지를 직접 폴링하는 대신, 컨슈머가 메시지를 자동으로 가져오도록 설정하고, 가져온 메시지를 특정 메서드로 전달하여 처리하는 방식입니다.

### 2-3. Consumer Group
`@KafkaListener` 파라미터를 보면 `groupId`가 있습니다. Consuemer Group은 `groupId`를 통해 지정됩니다. `groupId`는 컨슈머가 속한 컨슈머 그룹을 식별하는 고유 값으로, 여러 컨슈머가 같은 값을 공유하면 동일한 그룹에 속하게 됩니다. 

Consumer Group을 짧게 알아보자면 Kafka에서 **메시지를 병렬로 처리**하기 위해 사용하는 구조입니다. 하나의 토픽을 여러 컨슈머가 동시에 소비할 때, 각 컨슈머가 메시지를 중복해서 처리하지 않고, 토픽의 파티션별로 나누어 메시지를 처리할 수 있게 해줍니다. 같은 그룹에 속한 컨슈머들은 각자 토픽의 파티션을 할당받아 메시지를 나눠 처리합니다.

`groupId`는 ConsumerConfig 혹은 `application.yml`에 등록하여 처리할 수도 있습니다. 

컨슈머 그룹은 같은 토픽을 처리하는 것과 다른 토픽을 처리하는 것으로 분류할 수 있습니다. 

컨슈머 그룹의 가장 일반적인 사용 사례는 **하나의 토픽을 여러 컨슈머가 동시에 처리**하는 것입니다.

Kafka에서 **같은 컨슈머 그룹**에 속한 여러 컨슈머는 **토픽의 파티션을 나눠**서 메시지를 처리합니다. 하나의 토픽이 여러 파티션으로 나뉘어 있고, 같은 그룹에 속한 컨슈머들은 각 파티션을 할당받아 메시지를 처리합니다. 같은 그룹에 속한 컨슈머들끼리는 **메시지를 중복해서 처리하지 않도록** Kafka가 조정합니다. 같은 토픽을 여러 컨슈머가 병렬로 처리함으로써 **처리 성능**을 향상시킬 수 있습니다.

Kafka에서는 같은 컨슈머 그룹이 **여러 개의 다른 토픽을 동시에 구독하고 처리**할 수도 있습니다. 이 경우, 각 토픽의 메시지는 **독립적으로** 처리됩니다. 그러나 **컨슈머 그룹**의 의미가 퇴색되지는 않으며, 여전히 Kafka가 그룹 내에서 파티션을 적절히 할당하고 관리합니다.

Kafka는 **각 토픽별로 오프셋을 관리**하므로, 하나의 그룹 내에서 여러 토픽을 구독하더라도 토픽별로 메시지를 읽고 처리할 위치가 독립적으로 유지됩니다. 같은 그룹의 컨슈머가 **여러 토픽에서 들어오는 메시지를 병렬로 처리**할 수 있으므로, 메시지 처리 방식을 유연하게 관리할 수 있습니다.

컨슈머 그룹의 주된 목적은 **메시지를 병렬 처리**하고, **파티션별로 메시지를 분배**하여 **중복 처리를 방지**하는 것입니다. 여러 토픽을 같은 컨슈머 그룹에서 처리하더라도, **각 토픽의 파티션은 여전히 독립적으로 관리**됩니다. Kafka는 각 파티션에 대한 오프셋을 토픽별로 관리하기 때문에, **중복 처리는 방지**되고, 각 토픽에서 들어오는 메시지들이 **그룹 내 컨슈머들에게 적절히 분배**됩니다.

**🤔 그룹 내 같은 토픽을 받는 컨슈머를 만드려면 중복해서 리스너를 작성해야 하는가? 아니면 별도의 설정 값이 있는가?**
`concurrency` 설정을 통해 컨슈머의 수를 조절할 수 있습니다. 
```java
    factory.setConcurrency(3); // 동시에 3개의 스레드를 사용하여 메시지를 처리하도록 설정
```

### 2-4. 다양한 설정 값들 
ENABLE_AUTO_COMMIT_CONFIG :

우선 커밋이 무엇인지 파악해야 합니다. `Kafka`에서 커밋은 컨슈머가 메세지를 어디까지 읽었는지 그 정보를 (Offset) Kafka 브로커에 저장하는 방식입니다. 해당 오프셋 정보는 `Kafka`가 관리하는 특정한 장소에 저장됩니다. 

좀 더 세부적으로 들어가면 커밋된 오프셋 정보는 `__consumer_offsets` 이라는 내부 토픽에 저장됩니다. 

`true`로 설정하게 되면 반환된 메세지의 `offset`을 주기적으로 커밋합니다. 커밋된 `offset`은 컨슈머가 재시작되거나 실패 후 다시 작동할 때 마지막으로 처리된 위치에서 메세지를 읽어올 수 있게 합니다. 

`false`로 설정한다면 코드에서 명시적으로 오프셋을 커밋하는 시점을 정의해야 합니다. 일반적으로 메세지를 완벽하게 처리한 후에 수동으로 커밋하는 방법이 안전합니다. 

예제 코드는 다음과 같습니다. 
```java
public class KafkaConsumer {

    @KafkaListener(topics = "my-topic", groupId = "my-consumer-group")
    public void listen(String message, Acknowledgment acknowledgment) {
        try {
            // 메시지 처리 로직
            // 메시지 처리가 완료되면 오프셋 커밋
            acknowledgment.acknowledge();
        } catch (Exception e) {
	        // 예외 처리 
        }
    }
}

```

이 외에도 다양한 옵션이 존재합니다. 

`AUTO_COMMIT_INTERVAL_MS_CONFIG`: AUTO_COMMIT이 `true`일 때 자동으로 커밋하는 주기를 설정합니다. 기본 값을 5000ms 입니다. 

`MAX_POLL_RECORDS_CONFIG`: - 한 번의 `poll()` 호출로 가져오는 메시지의 최대 개수를 설정합니다. 기본값은 `500`입니다.

`SESSION_TIMEOUT_MS_CONFIG` : 컨슈머가 Kafka 브로커에 응답하지 않으면, 세션이 만료되어 컨슈머 그룹에서 제외됩니다. 이 타임아웃을 설정합니다.

`HEARTBEAT_INTERVAL_MS_CONFIG`: 컨슈머가 **Kafka 브로커**와 연결 상태를 유지하기 위해 보내는 하트비트 주기를 설정합니다. 이 값은 `SESSION_TIMEOUT_MS_CONFIG`의 3분의 1 이하로 설정하는 것이 일반적입니다.

`FETCH_MIN_BYTES_CONFIG`: 컨슈머가 서버로부터 데이터를 가져올 때, 최소한 가져올 데이터의 크기를 설정합니다. 설정된 크기만큼 데이터가 쌓이면 메시지를 가져옵니다.

`FETCH_MAX_WAIT_MS_CONFIG` : 설정된 `FETCH_MIN_BYTES_CONFIG`에 도달하지 않더라도, 지정된 시간 동안 대기한 후 데이터를 가져옵니다.

`MAX_PARTITION_FETCH_BYTES_CONFIG` : 컨슈머가 **파티션당** 한 번에 가져올 수 있는 최대 데이터 크기를 설정합니다. 기본값은 1MB(`1048576`).

`REQUEST_TIMEOUT_MS_CONFIG` : Kafka 브로커에 대한 요청이 실패했을 때 재시도하기까지 기다리는 최대 시간을 설정합니다.