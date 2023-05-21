### 개요

스프링 프로젝트를 진행하다보면 `@GetMapping`, `@PostMapping`, `@PutMapping`, `@RequestMapping`** 등 다양한 어노테이션을 써야하는 경우가 많습니다. 그 중에서도 `@GetMapping`과 `@PostMapping`의 차이에 대해 알아보려고 합니다.

### 두 어노테이션의 본질

두 어노테이션 모두 특정 URL을 수행할 컨트롤러와 매핑하는 역할을 해줍니다.

가령 다음과 같은 코드가 있다고 가정해봅시다.

```java
@RequestMapping(value = "/hello", method=RequestMethod.GET)
public String hello(Model model){
	model.addAttribute("hello", hello");
	return "hello";
}

@GetMapping("/hello"){
public String hello(Model model){
	model.addAttribute("hello", hello");
	return "hello";
}
```

두 주석 모두 [localhost:8080/hello](http://localhost:8080/hello) 로 들어오는 요청들을 해당 컨트롤러의 메소드와 연결해서 기능을 수행할 것입니다.

하지만 위 코드를 보면 두 메소드의 차이 점을 찾을 수 있을 겁니다. `@RequestMapping`은  method 인자를 통해 `GET` 방식이나 `POST` 방식 더 나아가 `PUT`, `PATCH`, `DELETE`를 지정하고 있습니다.

반면에 밑에 코드는 `@GetMapping`을 사용하면 비교적 짧은 코드로 의도를 명확히 나타낼 수 있습니다.

하지만 단지 짧은 코드가 근본적인 이유일까요?

### 근본적인 이유

`@GetMapping과` `@PostMapping`을 사용하면 url을 중복할 수 있습니다.

상품을 등록하는 짧은 예제 코드를 가져와 봤습니다.

```java
		@GetMapping(value = "/items/new")
    public String createForm(Model model) {
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    }

		@PostMapping(value = "/items/new")
    public String create(BookForm form) {
        Book book = new Book();
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/items";
    }
```

위 코드는 동일한 경로 하에 두 가지 메소드를 가지고 있습니다. `/items/new` 경로로 들어오게 되면 `Model` 객체에 "form"이라는 이름으로 `BookForm` 객체를 추가하고, "items/createItemForm" 뷰를 반환합니다. 이를 통해 `createItemForm.html` 뷰에서 폼을 작성할 수 있습니다.

또 작성이 다 된 폼을 `@PostMapping` 으로 처리함으로써 코드의 가독성과 유지보수가 용이해집니다.

하지만 `@RequestBody`는 중복된 URL을 사용할 수 없습니다.

물론 다음 코드와 같이 작성할 수 있습니다만 가독성이 떨어집니다.

```java
@RequestMapping(value = "/items/new", method = {RequestMethod.GET, RequestMethod.POST})
public String createOrUpdateForm(Model model, @ModelAttribute("form") BookForm form) {
    if (RequestMethod.GET.toString().equals(RequestContextHolder.currentRequestAttributes().getAttribute(HandlerMapping.BEST_MATCHING_PATTERN_ATTRIBUTE, RequestAttributes.SCOPE_REQUEST))) {
        // GET 요청일 경우
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    } else {
        // POST 요청일 경우
        Book book = new Book();
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/items";
    }
}
```