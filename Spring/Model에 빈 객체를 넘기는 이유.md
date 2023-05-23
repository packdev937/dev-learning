# Model에 빈 객체를 넘기는 이유

본 게시글은 인프런 김영한님의 JPA 활용 1 - 웹 어플리케이션 개발 강의를 참조하여 작성하였습니다.

### 개요

스프링 프로젝트를 진행하다보면 HTML 파일에 객체의 필드 값을 사전에 줘야 하는 상황이 종종 있습니다. 이 때 값이 들어있는 객체를 주는 것이 아닌 단지 객체만 생성해서 주는 경우 즉, 빈 객체를 생성해서 전달하는 경우가 있는데요 이에 대해 자세히 알아보고자 합니다.

### new 로 객체 생성

`ItemController.java`

```java
    @GetMapping(value = "/items/new")
    public String createForm(Model model) {
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    }
```

위 코드는 `/items/new`로 GET 요청이 들어왔을 때 실행되는 메서드입니다. 여기서 다루고자 하는 내용은 왜 model.addAttribute(”form”, new BookForm()); 에서 새로운 객체를 생성하고 이를 Model에 전달하는가 입니다.

이는 newBookForm()을 통해 폼을 초기화 하기 위해서 입니다.

`createItemForm.html`

```java
<form th:action="@{/items/new}" th:object="${form}" method="post">
```

html 파일에는 다음과 같은 타임리프 문법이 작성되어 있습니다. 이는 ‘form’ 이라는 이름으로 모델에 저장된 객체를 폼의 데이터 바인딩 대상으로 사용한다는 것입니다.

만약 ‘form’ 객체를 초기화 하지 않는다면 다른 방식으로라도 ‘form’ 객체를 초기화 해야 합니다. 그렇지 않으면 ‘form’ 객체가 없어 폼의 데이터 바인딩이 실패하거나 예외가 발생할 수 있습니다.

### 빈 객체를 넘기지 않는 경우

반대로 빈 객체를 넘기지 않는 경우 또한 존재합니다.

```java
    @GetMapping("items/{itemId}/edit")
    public String updateItemForm(@PathVariable("itemId") Long itemId, Model model) {
        Book item = (Book) itemService.findOne(itemId);

        BookForm form = new BookForm();
        form.setId(item.getId());
        form.setName(item.getName());
        form.setPrice(item.getPrice());
        form.setStockQuantity(item.getStockQuantity());
        form.setAuthor(item.getAuthor());
        form.setIsbn(item.getIsbn());

        model.addAttribute("form", form);
        return "items/updateItemForm";
    }
```

다음은 아이템 등록 정보를 변경하는 메소드 입니다. 해당 메소드의 경우는 itemId를 넘겨 받아, 해당 아이디로 저장되어 있는 아이템을 데이터베이스에서 불러온 뒤 수정합니다. 그리고 이번에는 `model.addAttribute("form", form)` 으로 ‘form’ 객체를 넘깁니다.

이는 구현하고자 하는 기능의 차이 때문에 발생하는 일입니다. 우리가 회원 가입을 하거나 어떤 물건을 새롭게 등록할 때, 기존의 정보는 아무 것도 입력되지 않은 상태여야 합니다. 따라서 빈 객체를 생성하여 넘깁니다.

반대로 우리가 어떤 정보를 수정하고자 할 때는 기존의 정보가 화면에 출력된 상태여야 합니다. 기존의 정보를 보고 내가 수정하고자 하는 필드의 값을 수정하는 형식입니다.

따라서 이러한 기능 차이가 form 객체로 무엇을 넘기는가에 대한 차이를 불러일으킵니다.