테스트 스텁이란 테스트 중인 모듈이 호출하는 다른 소프트웨어 구성 요소를 일시적으로 대체하는 소프트웨어 구성 요소를 의미합니다.

다음 그림은 Unit A가 Unit B에 의존하는 모습입니다. 만약 Unit B를 구현하기 힘들다면 이를 Stub으로 대체할 수 있습니다. 
![[Test Stub (1).png]]

### 예제 코드로 알아보는 Stub 
```java
private class OrderService {  
  
    private final OrderPort orderPort;  
  
    private OrderService(OrderPort orderPort) {  
        this.orderPort = orderPort;  
    }  
  
    public void createOrder(CreateOrderRequest request) {  
        final Product product = orderPort.getProductById(request.productId());  
  
        final Order order = new Order(product, request.quantity());  
  
        orderPort.save(order);  
    }  
}
```

위 코드는 OrderSerivce의 일부분입니다. OrderService에서는 주문 생성을 위하여 orderPort 내에 있는 getProductId()와 save()를 사용하고 있습니다. 

중요한건 OrderPort 내의 두 메소드가 의존하는 객체입니다. 
```java
public interface OrderPort {  
  
    Product getProductById(Long productId);  
  
    void save(Order order);  
}  
  
private class OrderAdapter implements OrderPort {  
  
    private final ProductRepository productRepository;  
    private final OrderRepository orderRepository;  
  
    private OrderAdapter(ProductRepository productRepository, OrderRepository orderRepository) {  
        this.productRepository = productRepository;  
        this.orderRepository = orderRepository;  
    }  
  
    @Override  
    public Product getProductById(final Long productId) {  
        return productRepository.findById(productId).orElseThrow();  
    }  
  
    @Override  
    public void save(final Order order) {  
        orderRepository.save(order);  
    }  
}
```

보시면 OrderPort를 상속받은 OrderAdapter가 각각 ProductRepository와 OrderRepository를 의존하는 것을 확인할 수 있습니다.

중요한건 ProductRepository는 현재 Jpa Repository로 구성되어 있습니다. 

이 때, 우리는 Test Stub을 사용할 수 있습니다. 
```java
@BeforeEach  
   void setUp() {  
       orderRepository = new OrderRepository();  
       orderPort = new OrderPort() {  
           @Override  
           public Product getProductById(Long productId) {  
return new Product("상품명", 1000, DiscountPolicy.NONE);  
           }  
  
           @Override  
           public void save(Order order) {  
orderRepository.save(order);  
           }  
       };  
       orderService = new OrderService(orderPort);  
   }
```

즉, OrderPort 내 메소드를 호출해서 반환 받은 값들을 임의로 정의해 놓는 것입니다. 

### 스텁이 사용되는 경우 
*"구현 되지 않은 함수나 라이브러리에서 제공되는 함수를 사용할 때"*

최근 개발 동향에 따르면 테스트는 개발과 동시에 진행됩니다. 이 때 의존하려는 객체가 개발이 완성되지 않은 경우가 존재할 수 있습니다. 이 때, Stub으로 해당 객체 혹은 메소드를 대체합니다.

*"함수가 반환하는 값을 임의로 생성하고 싶을 때"*

*"복잡한 논리 흐름을 가지는 경우에 테스트를 단순화 하고 싶을 때"*

*"의존성을 가지는 유닛의 응답을 묘사하여 독립적인 테스트를 수행 하고자 할 때"*

### Reference
- https://blog.naver.com/suresofttech/221180956096
- https://azderica.github.io/00-test-mock-and-stub/
	└ Stub과 Mock에 대한 예시인데 학습에 도움이 많이 됐습니다.
- https://medium.com/daangn/%ED%9A%A8%EC%9C%A8%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%9C%84%ED%95%9C-stub-%EA%B0%9D%EC%B2%B4-%ED%99%9C%EC%9A%A9%EB%B2%95-5c52a447dfb7
- 