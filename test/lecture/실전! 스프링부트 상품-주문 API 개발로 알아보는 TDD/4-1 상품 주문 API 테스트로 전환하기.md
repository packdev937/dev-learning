우선 스프링 부트 테스트로 구현한 상품 주문 API 기능입니다.
```java
@SpringBootTest  
class OrderServiceTest {  
  
    @Autowired  
    private OrderService orderService;  
  
    @Autowired  
    private ProductService productService;  
  
    @Test  
    void 상품주문() {  
        productService.addProduct(ProductSteps.상품등록요청_생성());  
        CreateOrderRequest request = 상품주문요청_생성();  
        orderService.createOrder(request);  
    }  
  
    private static CreateOrderRequest 상품주문요청_생성() {  
        final Long productId = 1L;  
        final int quantity = 2;  
        CreateOrderRequest request = new CreateOrderRequest(productId, quantity);  
        return request;  
    }  
}
```

앞선 기능 같이 REST-Assured를 사용해서 Api 테스트를 진행해줍니다. 

예를 들어, orderSerivce를 통해 직접 호출했던 메소드는 경로를 통해 호출해줍니다. 
```java
class OrderApiTest extends ApiTest {  
  
    @Test  
    void 상품주문() {  
        ProductSteps.상품등록요청(ProductSteps.상품등록요청_생성());  
  
        ExtractableResponse<Response> response = RestAssured  
            .given().log().all()  
            .contentType(MediaType.APPLICATION_JSON_VALUE)  
            .body(상품주문요청_생성())  
            .when()  
            .post("/orders")  
            .then().log().all()  
            .extract();  
  
        Assertions.assertThat(response.statusCode()).isEqualTo(HttpStatus.CREATED.value());  
    }  
  
    private static CreateOrderRequest 상품주문요청_생성() {  
        final Long productId = 1L;  
        final int quantity = 2;  
        CreateOrderRequest request = new CreateOrderRequest(productId, quantity);  
        return request;  
    }  
}
```