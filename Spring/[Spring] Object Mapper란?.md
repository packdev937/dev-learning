# [Spring] Object Mapper란?

### 개요

스프링을 공부하던 도중 실무에서 자주 사용한다는 `ObjectMapper` 를 알게 되었습니다. Java 객체를 JSON 객체로 또는 JSON 객체를 자바 객체로 직렬화 할 때 사용하는 클래스인데 조금 더 자세히 알아보고자 글로서 작성하게 되었습니다.

### Object Mapper란?

`ObjectMapper` 는 앞서 언급한 것 처럼 JAVA 객체를 JSON 객체로, JSON 객체를 JAVA 객체로 직렬화 혹은 역직렬화 할 때 사용하는 클래스 입니다.

### 어떤 경우에 사용할까요?

코드를 구현하다보면 간혹가다 자바 객체를 넣으면 작동이 안되는 코드가 몇몇 있습니다. 다음과 같은 예제인데요, 해당 코드는 `/posts` 경로로 `POST`가 요청되면, `content()` 안에 있는 JSON 이 넘어가는 구조입니다.

```java
@Test
void test() throws Exception {
mockMvc.perform(post("/posts")
                .contentType(APPLICATION_JSON)
                .content("{\"title\":\"제목입니다.\", \"content\" : \"내용입니다.\"}"))
            .andExpect(status().isOk())
            .andExpect(content().string("{}"))
            .andDo(print());
}				
```

저는 위 코드를 다음과 같이 바꾸고 싶었습니다.

```java
PostCreateDto request = new PostCreateDto("제목입니다.", "내용입니다.");

...
                .contentType(APPLICATION_JSON)
                .content(request)
```

바로 수 작업으로 JSON을 작성하는 대신 객체를 넣어서 확인하는 것입니다. 하지만 위 코드는 문제가 있었는데요, 바로 `content()` 안에는 JSON만 넣을 수 있다는 것이였습니다. 즉, 위 request 객체를 JSON으로 직렬화 해야겠죠.

### 직렬화

자바 객체를 JSON으로 바꾸기 위해서는 `writeValue()` 를 사용합니다. 여기에서는 JSON 객체를 String의 형태로 전달하니 `writeValueAsString()`을 사용할 수 있습니다.

```java
PostCreateDto request = new PostCreateDto("제목입니다.", "내용입니다.");
ObjectMapper objectMapper = new ObjectMapper(); 
String JSON = objectMapper.writeValueAsString(request);

...
                .contentType(APPLICATION_JSON)
                .content(request)
```

다음과 같이 작성한다면 문제 없이 코드가 정상 작동 됩니다!

### 역직렬화

직렬화를 알아봤으니, 역직렬화를 알아봐야겠죠. 예제는 동일하게 방금 넘겨주었던 밑의 JSON을 자바 객체로 역직렬화 해보도록 하겠습니다.

```java
{
		"title" : "제목입니다.",
		"content" : "내용입니다."
}
```

write의 반의어가 뭐죠? 바로 read입니다. 역직렬화 시에는 `readValue()` 메소드를 사용합니다.

예제는 다음 코드를 통해 알아볼 수 있습니다.