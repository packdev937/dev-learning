
WebClient를 사용해서 Naver Datalab API를 호출하는 코드 
``` java
public DatalabSearchResponse search(CharacterAnalysisRequest request) {  
    WebClient webClient = WebClient.builder()  
        .baseUrl(baseUrl)  
        .defaultHeader("X-Naver-Client-Id", clientId)  
        .defaultHeader("X-Naver-Client-Secret", clientSecret)  
        .build();  
  
    return webClient.post().uri(trendBaseUrl)  
        .contentType(MediaType.APPLICATION_JSON)  
        .bodyValue(DataLabRequest.of(convertToFormattedDate(request.startDate()), convertToFormattedDate(request.endDate()), request.keyword()))  
        .retrieve()  
        .bodyToMono(DatalabSearchResponse.class)  
        .block();  
}
```

Google Access Token을 발급 받는 코드
```java
    public AccessTokenResponse getAccessToken(String authCode) {  
        WebClient webClient = WebClient.create();  
  
        Mono<AccessTokenResponse> mono = webClient.post()  
            .uri(tokenUri)  
            .contentType(MediaType.APPLICATION_FORM_URLENCODED)  
            .body(BodyInserters.fromFormData("code", authCode)  
                .with("client_id", clientId)  
                .with("client_secret", clientSecret)  
                .with("redirect_uri", redirectUri)  
                .with("grant_type", "authorization_code"))  
            .retrieve()  
            .bodyToMono(JsonNode.class)  
//            .doOnNext(jsonNode -> log.info("Received JsonNode: " + jsonNode)) // JsonNode 확인  
            .map(jsonNode -> new AccessTokenResponse(jsonNode.get("access_token").asText(),  
                jsonNode.get("id_token").asText()));  
  
        return mono.block();  
    }
```