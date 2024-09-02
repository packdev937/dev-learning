## 1. SAGA 패턴이란?
> SAGA는 서비스 전반에 걸쳐 분산된 장기 실행 트랜섹션에 사용됩니다.

### 1-1. 장기 트랜섹션 
장기 실행 트랙센션이란 수명이 긴 트랙섹션입니다. 

장기 실행 트랙섹션의 예시는 다음과 같습니다.
- 온라인 쇼핑몰의 주문 처리
    - 사용자가 온라인 쇼핑몰에서 주문을 하면, 주문 처리 과정은 단일 트랜잭션으로 끝나지 않습니다. 예를 들어, 주문이 생성된 후 결제가 이루어지고, 재고 확인이 필요하며, 물류와 배송 절차가 차례로 진행됩니다. 이 모든 과정은 시간이 걸리며, 각 단계는 독립적으로 성공하거나 실패할 수 있습니다.
    - 만약 결제가 완료되었지만 재고가 부족한 경우, 결제를 취소하거나 대체 상품을 제안하는 등 다른 조치를 취할 수 있습니다. 이러한 시나리오에서는 각 단계별로 트랜잭션을 관리해야 하며, 상태 변경이 필요할 경우 롤백할 수 있어야 합니다.
- 여행 예약 시스템
    - 항공편, 호텔, 렌터카 예약이 필요한 복합적인 여행 예약 시스템에서는 여러 서비스 간의 연동이 필요합니다. 예를 들어, 사용자가 항공편을 예약한 후 호텔과 렌터카도 연달아 예약해야 하는 상황이 발생할 수 있습니다.
    - 항공편 예약은 성공했지만 호텔 예약이 실패하면, 시스템은 항공편 예약을 취소하거나 사용자가 다른 호텔을 선택하도록 안내해야 할 수 있습니다. 이처럼 여행 예약 시스템은 여러 단계에 걸쳐 트랜잭션이 진행되며, 모든 단계가 완료될 때까지 트랜잭션이 지속됩니다.
    - 항공편, 호텔, 렌터카 예약이 필요한 복합적인 여행 예약 시스템에서는 여러 서비스 간의 연동이 필요합니다. 예를 들어, 사용자가 항공편을 예약한 후 호텔과 렌터카도 연달아 예약해야 하는 상황이 발생할 수 있습니다.
    - 항공편 예약은 성공했지만 호텔 예약이 실패하면, 시스템은 항공편 예약을 취소하거나 사용자가 다른 호텔을 선택하도록 안내해야 할 수 있습니다. 이처럼 여행 예약 시스템은 여러 단계에 걸쳐 트랜잭션이 진행되며, 모든 단계가 완료될 때까지 트랜잭션이 지속됩니다.

SAGA의 기본 아이디어는 장기 실행 트랜섹션을 완료하기 위한 서비스 전반에 걸친 로컬 ACID 트랙섹션 체인을 만드는 것입니다.

### 1-2. 로컬 ACID 트랙섹션 체인 
트랜섹션 체인이란 각 로컬 트랙센션이 순차적으로 실행되어 전체 트랙섹션을 구성하는 것을 의미합니다. SAGA 패턴에서의 전체 트랜섹션은 각 서비스에서 발생하는 로컬 트랙섹션의 연속적인 체인이라고 이해하면 됩니다. 

ACID는 데이터베이스 트랜섹션 용어로 원자성, 일관성, 격리성, 내구성을 의미합니다. ACID 트랙섹션 체인이란 로컬 트랙섹션이 성공하면 다음 트랙섹션 체인을 실행하고 실패한다면 이전 트랙섹션을 롤백 하는 것을 의미합니다. 

## 2. SAGA 패턴 구성하기 
SAGA 패턴을 구성하는 방법은 여러 가지가 있지만 본 포스팅에서 다룰 내용은 Kafka를 이벤트 버스로 사용하는 SAGA 패턴입니다. 

예제는 앞 전에 언급했던 온라인 쇼핑몰의 주문 처리 과정으로 구성해보겠습니다. 

주문 처리 과정을 Aggregate로 나누면 주문 (Order), 결제 (Payment), 물류 (Product)로 나뉠 수 있습니다. 

주문 서비스는 SAGA의 흐름을 조정하는 역할을 합니다. 
SAGA를 시작하며 주문 생성 이벤트를 결제 서비스로 전송합니다. 해당 시점에서 주문 서비스의 로컬 디비에는 보류 중인 주문이 표시됩니다. 

그 다음 결제 서비스에서 결제 완료 이벤트를 가져오며 이 때의 주문 상태는 결제 완료 상태가 됩니다. 주문 서비스의 로컬 디비에 상태 값 또한 업데이트 해야겠죠. 

주문 서비스는 물류 서비스에 주문 승인 이벤트를 전송합니다. 물류 서비스에서 주문 승인 이벤트가 응답된다면 주문 서비스의 로컬 디비에 주문 상태를 주문 완료로 변경합니다.  

SAGA 패턴에서 중요한 것은 하나의 작업이 성공했을 때 다음 단계로 진행하는 행위, 그리고 실패했을 때 롤백하는 행위 입니다. 

## 3. 주문 서비스 
먼저 SAGA Step 인터페이스를 만들어줍니다. 
```java
public interface SagaStep<T, S extends DomainEvent, U extends DomainEvent> {

	S process(T data);
	U rollback(T data);
}
```

해당 인터페이스는 각 단계를 정의하는데 사용합니다. 

앞 전에 언급했다싶이 전체 트랙섹션 체인은 여러 개의 로컬 트랙섹션으로 나뉘게 됩니다. 각 로컬 트랙섹션이 성공했을 때 (process)와 실패했을 때 (rollback) 을 정의하기 위해 다음 인터페이스를 생성해야 합니다. 

### 3-1. SagaStep 
예시로는 다음과 같은 구현 클래스가 나올 수 있습니다. 
```java
@RequiredArgsConstructor
public class OrderPaymentSagaStep implements SagaStep<PaymentResponse, OrderPaidEvent, EmptyEvent> {

	private final OrderDomainService orderDomainService;
	private final OrderRepository orderRepository;
	private final OrderPaidProductRequestMessagePublisher orderPaidProductRequestMessagePublisher; 
	
    @Override
    public OrderPaidEvent process(PaymentResponse response) {
	    // Order 객체를 조회 
		Order order = orderRepository.findOrderById(response.getOrderId);
		// 주문 상태를 PAID로 변경 
        OrderPaidEvent orderPaidEvent = orderDomainService.payOrder(order);
        // 로컬 데이터베이스에 업데이트 
        orderRepository.save(order);
        return orderPaidEvent);
    }

    @Override
    public EmptyEvent rollback(OrderData data) {
	    // Order 객체를 조회 
		Order order = orderRepository.findOrderById(response.getOrderId);
		// Order 상태를 취소로 변경
		orderDomainService.cancel(order);
		// 이전 Step이 없으므로 별도의 Event 생성은 존재하지 않음 
        return EmptyEvent.INSTANCE;
    }
}
```

### ⭐️ 추가적인 Tip
객체를 조회하고 검증하는 `findXXX()` 메소드는 많은 클래스에서 참조하기 때문에 이를 Util 클래스로 분리 가능합니다. 클래스 명은 자유지만 여기에서는 Helper로 명명하겠습니다. 
```java
@RequiredArgsConstructor 
@Component
public class OrderSagaHelper {
	private final OrderRepository orderRepository; 

	public Order findOrder(String orderId) {
		Optional<Order> order = orderRepository.findById(orderId);

		if(order.isEmpty) {
			log.error("~");
			throw new OrderNotFoundException("~');
		}

		return order.get();
	}

	...
}
```

위 뿐만 아니라 다양한 Util 형 메소드가 Helper 클래스에 위치할 수 있습니다. 

### 3-2. MessageListener
위에서 구현한 Saga Step은 MessageListener에 의해 사용됩니다. 
```java
@RequiredArgsConstructor 
@Service 
public class PaymentResponseMessageListenerImpl implement PaymentResponseMessageListener {

	private final OrderPaymentSagaStep orderPaymentSagaStep;

	public void paymentCompleted(PaymentResponse paymentResponse) {
		OrderPaidEvent orderPaidEvent = orderPaymentSagaStep.process(paymentResponse);
		orderPaidEvent.fire();
	}
}
```

그리고 해당 MessageListener는 KafkaListener를 통해 사용됩니다.
```java
@KafkaListener(id = "${kafka-consumer-config.payment-consumer-group-id}", topics = "${order-service.payment-response-topic-name}")  
public void receive(@Payload List<PaymentResponseAvroModel> messages,  
                    @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) List<String> keys,  
                    @Header(KafkaHeaders.RECEIVED_PARTITION_ID) List<Integer> partitions,  
                    @Header(KafkaHeaders.OFFSET) List<Long> offsets) {  
            messages.size(),  
            keys.toString(),  
            partitions.toString(),  
            offsets.toString());  
  
    messages.forEach(paymentResponseAvroModel -> {  
        try {  
            if (PaymentStatus.COMPLETED == paymentResponseAvroModel.getPaymentStatus()) {                  paymentResponseMessageListener.paymentCompleted(paymentResponseAvroModel);  
            } else if (PaymentStatus.CANCELLED == paymentResponseAvroModel.getPaymentStatus(){
	            ...
            }  
        }
```

😒 왜 KafkaListener -> MessageListener -> SagaStep으로 스텝이 나누어질까요? 언뜻 보기에는 KafkaListener에서 바로 SagaStep을 호출할 수 있어 보입니다. 

이는 책임 분리의 원칙 때문입니다. 
KafkaListener는 메세지 수신의 역할을 하고, MessageListener는 비즈니스 로직을 담당합니다. 그리고 SagaStep은 SAGA의 특정 단계에 대한 비즈니스 로직을 담당하죠.

각 계층을 추상화 하였기에 MessageListener 혹은 SagaStep의 비즈니스 로직이 변경되어도 상위 계층의 클래스는 그 변경에 영향을 받지 않습니다.

실제로도 KafkaListener는 MessageListener를 의존하고 있으며 런타임에는 그 구현체 MessageListenerImpl이 사용됩니다. 