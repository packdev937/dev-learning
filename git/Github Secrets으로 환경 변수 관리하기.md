`application-prod.yml`
```yaml
spring: 
	config:  
	  import: "optional:classpath:secrets.yml"

openai: base-url: ${openai.base-url} api-key: ${openai.api-key} cloud: aws: s3: bucket: ${cloud.aws.s3.bucket} stack: auto: ${cloud.aws.stack.auto} region: static: ${cloud.aws.region.static} credentials: accessKey: ${cloud.aws.credentials.accessKey} secretKey: ${cloud.aws.credentials.secretKey} naver: client-id: ${naver.client-id} client-secret: ${naver.client-secret}
```



### Reference
- https://velog.io/@2ast/Github-Github-actions%EC%97%90%EC%84%9C-Secrets%EB%A1%9C-%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0