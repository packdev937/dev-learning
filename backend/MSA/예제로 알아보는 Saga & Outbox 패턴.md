## 1. 개요
오늘 포스팅에서는 Outbox 패턴에 대해 알아보겠습니다. Outbox 패턴은 왜 필요할까요?

Saga 패턴은 분산 시스템에서 **장기 실행 트랜잭션**을 관리하고, **데이터 일관성**을 유지하는데 유용한 패턴으로, 전체 트랜잭션을 각각의 로컬 트랜잭션으로 나누어 관리합니다. 각 로컬 트랜잭션이 성공적으로 완료되면 다음 단계로 진행하고, 실패할 경우 이전에 성공한 트랜잭션에 대해 **보상 작업**을 실행하여 전체 시스템의 일관성을 유지합니다.

Saga 패턴을 사용할 때 데이터베이스와 메시지 브로커 사이에 일관성을 보장하는 것은 여전히 문제가 됩니다. 예를 들어, 로컬 트랜잭션에서 데이터베이스 업데이트가 성공했지만, 해당 이벤트를 메시지 브로커에 발행하는 단계에서 실패한다면, 데이터베이스 상태와 이벤트 시스템 간의 불일치가 발생할 수 있습니다.

**Outbox 패턴**은 **트랜잭션**이 성공적으로 커밋된 후에만 외부 시스템으로 메시지가 전송되도록 보장합니다. 이를 위해 이벤트를 먼저 **Outbox 테이블**에 저장하고, 트랜잭션이 성공적으로 완료된 후 외부 시스템으로 메시지를 전송합니다.

따라서 Saga와 Outbox 패턴을 결합하여 일관된 솔루션을 구축해야 합니다. **Outbox 패턴**은 **이벤트 발행** 시 데이터의 일관성을 보장하고, **Saga 패턴**은 여러 서비스 간의 트랜잭션을 관리하며, 전체 흐름에서 **보상 작업**을 처리합니다. 예를 들어, **Saga 패턴**에서 하나의 서비스가 트랜잭션을 성공한 후 이벤트를 발행할 때, **Outbox 패턴**을 사용하여 이벤트의 신뢰성을 보장할 수 있습니다.
## 2. 실습하기 
가장 크게 **고민했던 기능**의 **실행 흐름**은 다음과 같습니다.

사용자가 네컷 사진의 QR 코드를 인식하면, 해당 QR 코드에 포함된 URL에서 이미지를 추출한 후, 해당 이미지를 AWS S3에 저장하는 과정이 먼저 진행됩니다. 이후 사용자는 저장된 이미지를 바탕으로 제목, 한 줄 소개, 태그할 팔로워 등의 정보를 작성합니다. 이 과정에서 새로운 `Photo` 객체가 생성됩니다.

이제 작성자가 태그한 팔로워들을 바탕으로, 각 팔로워의 피드에 해당 사진이 나타나도록 `Feed` 객체가 생성됩니다. `Photo`와 `Feed`는 1: N 관계를 가지며, 한 `Photo` 객체가 여러 `Feed` 객체를 참조합니다. 이때 `Feed` 객체는 각 팔로워의 정보와 함께 생성되며, `Photo` 객체의 메타데이터를 포함하고 있습니다.

이 과정에서 중요한 점은 **트랜잭션의 일관성**입니다. 만약 `Feed` 객체의 생성 과정에서 어떤 이유로 실패가 발생한다면, 관련된 `Photo` 객체의 생성도 함께 롤백되어야 합니다. 이를 통해, 일부 팔로워에게만 피드가 생성되는 상황을 방지하고, 데이터의 무결성을 보장할 수 있습니다.

### ❗️ 실습 전 알아야 할 개념 
`@TransactionalEventListener`는 스프링 프레임워크에서 제공하는 애노테이션으로, **트랜잭션의 상태**에 따라 이벤트를 처리할 수 있도록 해주는 기능입니다. 주로 트랜잭션이 성공적으로 커밋되거나 롤백된 후에 특정 작업을 처리할 때 사용됩니다. 이 애노테이션을 사용하면, 트랜잭션의 상태에 맞춰 이벤트 핸들러 메서드를 호출할 수 있습니다.

`TransactionPhase`: 이 애노테이션은 트랜잭션이 어느 단계에서 이벤트를 처리할지 결정할 수 있도록 `phase`라는 속성을 제공합니다. 대표적인 단계는 다음과 같습니다:
    
`BEFORE_COMMIT`: 트랜잭션이 커밋되기 **직전**에 이벤트를 처리합니다.
 `AFTER_COMMIT`: 트랜잭션이 **성공적으로 커밋된 후** 이벤트를 처리합니다.
`AFTER_ROLLBACK`: 트랜잭션이 **롤백된 후**에 이벤트를 처리합니다.
`AFTER_COMPLETION`: 트랜잭션이 완료된 후(성공 여부와 상관없이) 이벤트를 처리합니다.

```java
@Component
public class PhotoEventListener {

    // 트랜잭션이 성공적으로 커밋된 후에 실행
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handlePhotoCreatedEvent(PhotoCreatedEvent event) {
        // 카프카에 이벤트를 발행하거나 다른 비동기 작업을 수행
        publishToKafka(event);
    }
    
    // 트랜잭션 롤백 후에 실행
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void handleRollback(PhotoCreatedEvent event) {
        // 롤백 후 처리할 작업
        log.warn("Photo creation rolled back. Event: {}", event);
    }
}
```

해당 어노테이션을 알아야 하는 이유는 **Outbox 패턴**에서 중요한 트랜잭션 절차에 맞춰 이벤트 처리 흐름을 제어할 수 있기 때문입니다.

### 2-1. 기존의 로직
기존의 로직에서 몇 가지 문제점을 찾아보겠습니다.
```java
   @Transactional  
   public CreatePhotoResponse createPhoto(CreatePhotoRequest createPhotoRequest) {  
  
       List<String> userIds = addUploaderToTaggerUsers(createPhotoRequest);  
  
       Photo photo = photoDataMapper.toDomain(createPhotoRequest, userIds);  
       Photo savedPhoto = photoRepository.save(photo);  
  
       publishCreatedPhotoEvent(savedPhoto);  
  
       if (savedPhoto == null) {  
           log.error("Could not create photo. Request User ID : {}",  
createPhotoRequest.requestUserId());  
  
           throw new PhotoNotCreatedException();  
       }  
  
       log.info("Returning CreatePhotoRequest for photo. Request User ID : {}",  
           createPhotoRequest.requestUserId());  
  
       return photoDataMapper.toCreateResponse(  
           savedPhoto.getPhotoId(),  
           "포토가 정상적으로 생성되었습니다."  
       );  
   }
```

첫 번째로 트랙센션이 강결합되어 있습니다. 즉, 데이터베이스에 저장하는 행위와 메세지큐에 이벤트를 발행하는 행위를 동시에 처리하려고 하고 있습니다. 이는 하나의 행위가 실패했을 때 전체 작업을 롤백 해야 하는 문제가 있습니다. 이는 시스템의 전체 효율성의 저하로 이어집니다. 

따라서 `@TransactionalEventListener` 등을 사용하여, 데이터베이스 트랜잭션이 성공적으로 커밋된 후에만 외부 시스템으로 이벤트를 발행하도록 변경해야 합니다. 이렇게 하면 데이터베이스는 안전하게 업데이트되고, 이벤트 발행은 별도의 비동기 작업으로 처리되어 **트랜잭션 일관성**을 유지하면서도 외부 시스템 실패로 인한 문제를 줄일 수 있습니다.

🤔 둘의 작업을 분리하더라도 메세지 발송이 실패하면 똑같이 롤백해야되는거 아닌가?
우리가 알아보고자 하는 **Outbox 패턴**을 사용하여 이를 해결할 수 있습니다.

1. Outbox 패턴은 도메인 이벤트를 Outbox 테이블에 기록합니다. 기록하는 행위는 데이터베이스의 트랙섹션 내에서 실행됩니다. 
2. 트랜잭션이 성공적으로 커밋되면, 별도의 비동기 프로세스가 Outbox 테이블에서 기록된 이벤트를 읽고 외부 시스템(예: Kafka)으로 메시지를 발행합니다.
3. 외부 시스템으로 이벤트 발행이 실패하면, 해당 이벤트는 Outbox 테이블에 그대로 남아 있기 때문에 **재시도** 메커니즘을 통해 나중에 다시 시도할 수 있습니다.

즉, 이전에 문제가 되었던 데이터베이스 트랙섹션은 실행되고 메세지 발행은 실패하는 경우, 이제는 Outbox 테이블에 기록된 이벤트를 계속해서 재시도 함으로써 일관성을 유지할 수 있습니다. 

## 2-2. 로직을 재구성하기 
### 2-2-1. Photo-Service 에서 `PhotoCreatedExternalEvent` 발행 
해당 문제를 해결하기 위해 Saga 패턴과 Outbox 패턴을 결합하여 기능을 개발하였습니다. 우선 Photo-Service 에서 Photo 객체가 생성될 때, `PhotoCreatedExternalEvent` 가 발행됩니다. 해당 이벤트의 발행은 `PhotoEventService` 가 담당하고 있습니다.

```java
@Transactional  
public CreatePhotoResponse createPhoto(CreatePhotoRequest createPhotoRequest) {  
  
    List<String> userIds = addUploaderToTaggerUsers(createPhotoRequest);  
  
    Photo photo = photoDataMapper.toDomain(createPhotoRequest, userIds);  
    Photo savedPhoto = photoRepository.save(photo);  
  
  
    if (savedPhoto == null) {  
        log.error("Could not create photo. Request User ID : {}", createPhotoRequest.requestUserId());  
        throw new PhotoNotCreatedException();  
    }  
  
    photoEventService.publishEvent(PhotoCreatedExternalEvent.of(savedPhoto));
```

`PhotoEventService` 는 `ApplicationEventPublisher` 를 통해 해당 PhotoCreatedExternalEvent 이벤트를 발행하여 관련된 리스너들이 해당 이벤트를 처리할 수 있게 만듭니다.
```java
public class PhotoEventService {  
  
    private final ApplicationEventPublisher eventPublisher;  
  
    public void publishEvent(ExternalEvent event) {  
        log.info("Publishing event : {} at PhotoEventService", event);  
        eventPublisher.publishEvent(event);  
    }  
}
```

관련된 리스너는 총 두개로 `PhotoEventRecorder` 와 `PhotoEventMessenger` 입니다.

`PhotoEventRecorder` 는 `Outbox 테이블`에 해당 이벤트를 기록하는 책임을 갖고 있습니다. 이 때, 해당 리스너에는 `@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)` 다음 어노테이션이 붙어있습니다. 

우선 `BEFORE_COMMIT`을 사용하면 **데이터베이스에 포토 객체가 먼저 저장**되고, 트랜잭션이 커밋되기 **직전에** Outbox 테이블에 이벤트가 기록되는 순서를 강제합니다.

또한, 트랙섹션이 실패하거나 롤백하는 경우에 `BEFORE_COMMIT` 단계의 이벤트가 기록되지 않습니다.

이를 통해 이벤트 처리와 데이터 저장을 명확하게 분리하여 트랙섹션 커밋 시점에만 Outbox 테이블에 이벤트가 저장되도록 하였습니다.

```java
public class PhotoEventRecorder {  
  
    private final OutboxEventRepository outboxEventRepository;  
    private final ObjectMapper objectMapper;  
  
    @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)  
    public void recordEvent(PhotoCreatedExternalEvent photoCreatedExternalEvent) {  
        String payload = createPayload(photoCreatedExternalEvent);  
  
        PhotoOutboxEntity outboxEvent = PhotoOutboxEntity.builder()  
	        ...
            .build();  
  
        outboxEventRepository.save(outboxEvent);  
    }
```


**PhotoEventMessenger**는 `Outbox` 테이블에 저장된 이벤트 중 처리되지 않은 이벤트(즉, `OutboxStatus`가 `STARTED`인)를 지속적으로 조회하여, 이를 외부 메시지 큐로 전송하는 역할을 하고 있습니다. Messenger 에는 `TransactionPhase.AFTER_COMMIT` 속성이 설정되어 있습니다. 이를 통해 트랙섹션이 완료되지 않은 상태에서 외부 메세지 큐로 전송되는 것을 방지하고, 데이터 일관성을 유지할 수 있습니다.

**PhotoEventMessenger**는 외부 메시지 큐인 **KafkaPublisher**에 이벤트 발행을 요청합니다. **PhotoCreatedEventKafkaPublisher**는 매개변수로 `Outbox` 엔티티를 전달받아, 해당 엔티티에서 직렬화된 이벤트(**payload**)를 추출한 뒤 **Kafka** 서버로 이벤트를 발행합니다. 이 과정에서 발행이 성공하면 `OutboxStatus`는 `COMPLETED`로 업데이트되며, 발행에 실패할 경우 `FAILED`로 상태가 변경됩니다.

만약 **Kafka 전송** 중 문제가 발생하면, **Outbox 테이블에 이벤트가 남아** 있기 때문에 **재시도 메커니즘**을 적용할 수 있습니다. 이를 통해 이벤트 전송 실패 시에도 시스템이 재시도를 할 수 있습니다.

```java
@Component  
public class PhotoEventMessenger {  
  
    private final OutboxEventRepository outboxEventRepository;  
    private final PhotoCreatedEventKafkaPublisher photoCreatedEventKafkaPublisher;  
  
    @Async  
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)  
    public void sendEvent(PhotoCreatedExternalEvent event) {  
        List<PhotoOutboxEntity> outboxes = outboxEventRepository.findByOutboxStatusAndEventType(  
            OutboxStatus.STARTED, event.getEventType()  
        );  
  
        outboxes.addAll(outboxEventRepository.findByOutboxStatusAndEventType(  
            OutboxStatus.FAILED, event.getEventType()  
        ));  
  
        for (PhotoOutboxEntity outbox : outboxes) {  
            try {  
                photoCreatedEventKafkaPublisher.publish(  
                    outbox,  
                    this::updateOutboxStatus  
                );  
                outbox.updateOutboxStatus(OutboxStatus.COMPLETED);  
                outbox.updateSagaStatus(SagaStatus.PROCESSING);  
                outboxEventRepository.save(outbox);  
            } catch (Exception exception) {  
                outbox.updateOutboxStatus(OutboxStatus.FAILED);  
                outbox.updateSagaStatus(SagaStatus.FAILED);  
                outboxEventRepository.save(outbox);  
            }  
        }  
    }

	...
```

### 2-2-2. Feed-Service에서 `PhotoCreatedExternalEvent` 수신 및 피드 생성
**Feed-Service**의 **PhotoCreatedExternalEventKafkaListener**는 **Kafka** 서버로부터 이벤트를 수신합니다. 이 서비스에서는 피드 생성 중 실패가 발생할 경우, **Saga 패턴**을 활용하여 보상 작업을 수행해 이전 상태로 **롤백**하거나 **실패를 기록**해야 합니다.

수신된 이벤트(**payload**)는 먼저 `PhotoCreatedExternalEvent`로 역직렬화됩니다. 피드 생성을 위한 속성들은 해당 이벤트에서 추출할 수 있으며, 이를 처리하기 위해 **PhotoCreatedMessageListener**로 이벤트가 전달됩니다.
```java
public class PhotoCreatedExternalEventKafkaListener implements  
    KafkaConsumer<PhotoCreatedExternalEvent> {  
  
    private final PhotoCreatedMessageListener photoCreatedMessageListener;  
    private final ObjectMapper objectMapper;  
  
    @KafkaListener(topics = "photo_created_topic", containerFactory = "kafkaListenerContainerFactory")  
    @Override  
    public void receive(String message) {  
  
        try {  
            PhotoCreatedExternalEvent event = objectMapper.readValue(message,  
                PhotoCreatedExternalEvent.class);  
            log.info("Received PhotoCreatedExternalEvent: {} at PhotoCreatedExternalEventKafkaListener", event);  
  
            photoCreatedMessageListener.handleEvent(event);  
        } catch (Exception exception) {  
            log.error("Failed to deserialize JSON message to PhotoCreatedExternalEvent", exception);  
        }  
    }  
}
```

**PhotoCreatedMessageListener**는 적절한 **Saga 패턴**을 찾아 이벤트를 다시 전달합니다. 이 과정에서, try-catch 문을 사용하여 정상적으로 처리되면 Saga 클래스의 `process()`가 호출되고, 예외가 발생하면 `rollback()`이 호출됩니다.

```java
public class PhotoCreatedMessageProcessor implements PhotoCreatedMessageListener {  
  
    private final FeedCreationSaga feedCreationSaga;  
  
    @Override  
    public void handleEvent(PhotoCreatedExternalEvent event) {  
  
        log.info("Handling event for creating feeds: {} at PhotoCreatedMessageProcessor", event);  
  
        try {  
            feedCreationSaga.process(event);  
        } catch (Exception exception) {  
            log.error("Failed to process feed creation at PhotoCreatedMessageProcessor", exception);  
            feedCreationSaga.rollback(event);  
        }  
    }  
}
```

`process()`가 호출되면, 이벤트로 전달된 `Photo`의 속성을 기반으로 `Feed`가 생성됩니다. 반대로 `rollback()`이 호출되면, 이전 작업들을 취소하는 보상 트랙섹션이 실행됩니다.

만약 피드 생성 도중 **실패**한 경우, 이미 데이터베이스에 기록된 임시 데이터나 중간 생성된 리소스를 삭제합니다. 또한, 보상 트랜잭션이 실행되면 `PhotoCancelledExternalEvent`가 발행됩니다. 이 이벤트는 다른 시스템에 **이전 작업이 취소되었음을 알리는** 역할을 합니다. 이를 통해 `Feed` 생성 실패에 따른 **연관 서비스**들이 해당 취소 상태를 인지하고 적절한 후속 조치를 진행합니다. 이 경우는 **Photo-Service**에서 생성된 `Photo` 객체를 다시 롤백합니다.

## Reference
- https://medium.com/@greg.shiny82/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%94%EB%84%90-%EC%95%84%EC%9B%83%EB%B0%95%EC%8A%A4-%ED%8C%A8%ED%84%B4%EC%9D%98-%EC%8B%A4%EC%A0%9C-%EA%B5%AC%ED%98%84-%EC%82%AC%EB%A1%80-29cm-0f822fc23edb