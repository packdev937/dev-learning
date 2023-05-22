# @RequestBody 그리고 @ResponseBody

### 개요

웹 프로그래밍에서 JSON 통신은 매우 중요한 요소입니다. JSON은 데이터 교환 형식 중 하나이며 웹 애플리케이션에서 클라이언트와 서버 간의 데이터 교환에 많이 사용됩니다. 이러한 JSON 데이터들은 일반적으로 HTTP 요청 또는 응답의 바디에 포함되어 전달됩니다. 그렇기 때문에 **`@RequestBody`** 그리고 **`@ResponseBody`**를 잘 이해하고 적재적소에 활용하는 것은 매우 중요합니다. 이를 짧은 코드를 통해 학습해보고자 합니다.

### @RequestBody

해당 애노테이션이 붙은 매개변수에 HTTP 요청의 Body가 그대로 전달됩니다.

```java
@PostMapping("/api/books")
public ResponseEntity<String> createBook(@RequestBody BookRequest bookRequest) {
    Book newBook = new Book();
    newBook.setTitle(bookRequest.getTitle());
    newBook.setAuthor(bookRequest.getAuthor());
    
		// 비즈니스 로직 작성 
    return ResponseEntity.ok("Book created successfully");
}
```

위 코드에서는 Body를 통해 전송된 JSON 데이터를 bookRequest 객체로 변환하여 받고 있습니다.

**어떻게 변환을 받는가**

JSON 데이터는 {”키” : “값”} 으로 구성되어 있습니다. 위 예제에서 JSON 데이터가 {”title” : “Harry Potter”, “author” : “J.K. Rowling”} 으로 구성되어 있다고 가정해봅시다. BookRequest의 title 필드와 author 필드가 JSON 데이터들을 각각 매핑받습니다. BookRequest에 저장된 값들을 활용하여 다양한 비즈니스 로직을 구성할 수 있습니다.

**어떤 경우에 쓰는가**

마찬가지로 위 코드를 예로 들어 설명해보겠습니다. 위 코드는 전달받은 JSON 데이터를 bookRequest 객체에 매핑시킴으로써 Book에 대한 필드 값들을 설정할 수 있도록 구성하였습니다.

Book 객체를 새로 만들고 전달받은 필드 값들을 세팅하면 해당 객체를 통해 값을 저장할 수도, 혹은 수정을 지시할 수도 있습니다. 또한, 데이터의 유효성 검사도 진행할 수 있습니다. 여기서 중요한 포인트는 클라이언트가 전달한 값을 이용하여 작업을 수행한다는 점입니다.

### @ResponseBody

```java
@GetMapping("/api/books/{id}")
@ResponseBody
public ResponseEntity<Book> getBook(@PathVariable Long id) {
    Book book = bookService.getBookById(id);

    return ResponseEntity.ok(book);
}
```

위 코드에서는 `@ResponseBody`를 통해 요청한 ID에 해당하는 책을 데이터베이스에 조회한 뒤 클라이언트에 반환하고 있습니다.

**@RequestBody**와 차이점이 뭔가

우선 `@ResponseBody`는 말그대로 Body에 JSON 데이터를 실어서 응답을 보내는 것입니다. 그렇기 때문에 자연스럽게 `@GetMapping`과 함께 쓰여야합니다.

짧은 코드 몇 개를 통해 `@RequestBody` 그리고 `@ResponseBody`에 대해 알아보았습니다. 항상 어떠한 요청을 처리하기 전, 클라이언트가 JSON 데이터를 전달하는가/전달받는가에 대해 잘 고민하고 애노테이션을 사용하면 좋을 것 같습니다.