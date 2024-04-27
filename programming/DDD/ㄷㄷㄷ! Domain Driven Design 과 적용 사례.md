### 1. 개요 
[if(kakao) 2022](https://www.youtube.com/watch?v=4QHvTeeTsj0&t=778s) 발표를 보고 정리한 내용입니다. 
### 2. 코드로 접해보는 Haxagonal Architecture와 DDD 
![[스크린샷 2024-04-25 오후 2.30.48.png]]

`Product.java` 
```java
public class Product {
	private ProductSalesInfo productSalesInfo;
	private ProductMetaInfo productMetaInfo;
	private Content content;
	...
	// 판매 시작 
	public void startSale(LocalDateTime startSaleDate){
		validateProduct();
		productSalesInfo = ProductSalesInfo.startSale(startSaleDate);
	}
}
```

필드 값
- 작품에 대한 판매 정보
- 작품에 대한 메타 정보
- 작품이 가진 Content 파일 정보

도메인 로직
- 작품을 판매 시작 할 경우 유효성 검사 
- 판매 시작 날짜를 처리  

![[스크린샷 2024-04-25 오후 2.38.02.png]]
`ProductController.java`
```java
@PostMapping(path = "/product/{productId}")
public void saleProduct(@PathVariable("productId") Long productId){
	productUseCase.saleProduct(ProductSaleCommand
							.builder()
							.productId(new ProductId(productId))
							.build());
}

public interface ProductUseCase {
	void saleProduct(ProductSaleCommand command);
}
```

유저는 컨트롤을 통해 작품 판매 시작 API 로 요청을 합니다.

해당 API는 유스케이스에 정의된 작품 판매를 수행하게 됩니다. 
(유스 케이스는 사용자 기능 시나리오) 
유스 케이스에 정의된 saleProduct() 메소드는 `SaleService`에서 구현됩니다.

```java
public class SaleService implements ProductUseCase {
	private final LoadProductPort loadProductPort;
	private final SaveProductPort saveProductPort;
	private final EventPublishPort eventPublishPort;

	@Override
	public void saleProduct(ProductSaleCommand cmd){
		Product product = 
			loadProductPort.loadProduct(cmd.getProductId));
		product.startSale(LocalDateTime.now());

		saveProductPort.saveProduct(product);
		eventPublishPort.publisherEvent(product);
	}
}
```

작품을 판매할 때 조회하기 위해 LoadProductPort가 필요합니다.
조회한 Product에 대한 수정이 일어나면 SaveProductPort를 통해 저장합니다.
정리된 내용을 요약해서 알림 시스템으로의 메세지 전송을 위한 EventPublishPort를 사용하게 됩니다.
##### Load와 Save를 나누어 놓은 이유?
CRUD 중 조회를 제외한 나머지는 SavePort로 가고 있고 이는 CQRS 패턴. 즉, 커맨트와 쿼리 책임을 분리하기 위해서 입니다. 
또한 추후에 조회 성능 개선을 위해 Amazon SQS나 RabbitMQ 등을 사용하여 조회를 이벤트 드리븐 형식으로 변경할 수도 있기 때문입니다.

```java
class ProductPersistanceAdapter implements LoadProductPort, SaveProductPort {

	@Override
	public Product loadProduct (ProductId productId){
		ProductJpaEntity productJpaEntity = productRepository
			.findById(productId.getValue());
		ContentJpaEntity contentJpaEntity = contentRepository
			.findById(productId.getValue());
		return mapper.mapToDomainEntity(...);
	}

	@Override
	public void saveProduct(Product product) {
		productRepository.save(
				mapper.mapToJpaEntity(product));
	}
}
```

🤔 어댑터에서 오버라이드를 진행하면 interface를 상속받았을 때 어댑터에 구현된 로직을 바탕으로 실행되는가?

##### Mapper가 필요한 이유
Product 도메인을 JpaEntity로 혹은 JpaEntity를 Product라는 도메인으로 변경해주어야 하는데 이 때 mapper가 사용됩니다. 

이는 하나의 Aggregate는 여러 개의 테이블 혹은 데이터와 연관있을 수 있기 때문에 각각의 데이터를 하나의 도메인 집합으로 만들어 줄 때 변환하는 방식이 필요합니다. 
```java
class ProductMapper {
	Product mapToDomainEntity(...) {
		return Product.withId(
				new Product.ProductId(productJpaEntity.getId()),
				ProductSalesInfo...
				ProductMetaInfo...
				Content...);
	}

	ProductJpaEntity mapToJpaEntity(Product product) {
		return ProductJpaEntity...
			.build();
	}
}
```

