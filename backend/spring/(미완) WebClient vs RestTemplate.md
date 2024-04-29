### WebClient
**의존성** 
**WebClient 코드** 
```
public String chat(ChatMessageRequest request) {  
    WebClient webClient = WebClient.builder()  
        .baseUrl(baseUrl)  
        .defaultHeader("Authorization", "Bearer " + apiKey)  
        .build();  
  
    ChatMessageResponse response = webClient.post().uri(chatCompletionPostfix)  
        .contentType(MediaType.APPLICATION_JSON)  
        .bodyValue(ChatCompletionRequest.of(request.message()))  
        .retrieve()  
        .bodyToMono(ChatMessageResponse.class)  
        .block();  
  
    return response.getResponseMessage();  
}
```

### RestTemplate