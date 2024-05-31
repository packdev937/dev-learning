## 1. 개요
![[스크린샷 2024-05-31 오후 2.31.19.png]]

반복되는 Conntection is Closed를 어떻게 해결했는지 알아봅시다.

## 2. 트러블 슈팅
우선 ElasticSearchConfig 코드를 함께 살펴보겠습니다.
```java
@RequiredArgsConstructor  
@Configuration  
@EnableElasticsearchRepositories(basePackages = "packdev937.elasticsearch.repository")  
public class ElasticSearchConfig extends ElasticsearchConfiguration {  
  
    private final ElasticSearchProperties elasticSearchProperties;  
  
    @Override  
    public ClientConfiguration clientConfiguration() {  
        return ClientConfiguration.builder()  
            .connectedTo(elasticSearchProperties.getHost())  
            .withBasicAuth(elasticSearchProperties.getUsername(),  
                elasticSearchProperties.getPassword())  
            .build();  
    }  
}
```

설정 파일에서는 conntedTo에 주소를 넣어줘야 했는데요, 초기의 설정에서는 https://localhost:9200 으로 접근이 가능하여 해당 주소 값을 넣어줬습니다. 

```java
        return ClientConfiguration.builder()  
            .connectedTo(elasticSearchProperties.getHost())  
            .usingSsl()
            ...
```

하지만 그렇게 작성하다 보니 인증서를 전송하는 코드 또한 같이 작성해주어야 했습니다. 따로 인증서를 발급 받은 것도 아니고 Elastic Search가 자동으로 HTTPS를 설정해준건데 이는 아닌 것 같아 다시 HTTP로 연결을 시도했습니다.

하지만 http://localhost:9200 또한 작동을 하지 않았는데요, 저는 문제의 원인이 여기 있다고 생각하고 이를 해결하려고 했습니다.

문제는 쉽게 찾을 수 있었는데, elasticsearch.yml 파일 내에 다음 코드가 문제였습니다.
```
xpack.security.http.ssl.enabled: true
```

저는 이를 false로 바꿔주었고 그 결과 http://localhost:9200 이 정상 실행되는 것을 확인할 수 있었습니다. 
