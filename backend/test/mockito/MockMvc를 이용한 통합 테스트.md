예제 코드는 다음과 같습니다.
```java
@WebMvcTest(UserController.class)
class UserControllerIntegrationTest {

	// 가상의 HTTP 요청을 만들어 컨트롤러를 테스트 합니다.
	@Autowired
	private MockMvc mockMvc;

	private ObjectMapper objectMapper;

	@MockBean
	private UserApplicationUseCase userApplicationUseCase;

	@BeforeEach
	void setUp() {
		/*
		mockMvc = MockMvcBuilders
		.standaloneSetup(
			new UserController(userApplicationUseCase)
		)
		.build();
		*/

		objectMapper = new ObjectMapper();  
		// LocalDate, LocalDateTime 등을 직렬화/역직렬화 가능  
		objectMapper.registerModule(new JavaTimeModule());
	}

	@Test  
	void 유저를_생성한다() throws Exception {  
	    // Given  
	    String id = "packdev937";  
	    String message = "가입이 완료 되었습니다.";  
	    
	    CreateUserRequest createUserRequest = new CreateUserRequest(
		    id, 
		    "admin",  
		    LocalDate.of(1999, 3, 27)
		);
		  
	    CreateUserResponse createUserResponse = new CreateUserResponse(
		    id,  
	        message
	    );  
	  
	     //When & Then 
	     MvcResult mvcResult = mockMvc
		     .perform(post("/users") 
		     .contentType(MediaType.APPLICATION_JSON) 
		     // UTF-8 추가 
		     .characterEncoding("UTF-8") 
		     .content(objectMapper.writeValueAsString(createUserRequest))) 
		     .andExpect(status().isCreated()) 
		     .andExpect(jsonPath("$.userId").value(id)) 
		     .andExpect(jsonPath("$.message").value(message)) 
		     .andReturn();
	     
	     String response = mvcResult
	     .getResponse()
	     .getContentAsString(StandardCharsets.UTF_8); assertEquals(objectMapper
	     .writeValueAsString(createUserResponse), response); 
	     
	     // 예상한 횟수 만큼 호출, 메소드 호출 여부, 호출된 인자 등을 확인 
	     verify(userApplicationUseCase, times(1))
		     .createUser(
			     any(CreateUserRequest.class)
		     ); 
	     }
}
```
## Reference
- https://yuna-story.tistory.com/154
- https://yuna-story.tistory.com/155