압축된 ZIP 파일을 Spring으로 넘겨줘야 했는데 다음과 같은 에러가 발생했습니다.

우선 dependency를 추가해주어야 합니다. 
``` bash
implementation 'commons-io:commons-io:2.11.0'    /* Apache commons-io */
implementation group: 'commons-fileupload', name: 'commons-fileupload', version: '1.4'
```

그리고 몇몇 블로그에는 Bean을 등록하는 것을 확인할 수 있는데,
```java
@Bean 
public CommonsMultipartResolver multipartResolver() {
		CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver(); 
		// 파일 인코딩 설정 
		multipartResolver.setDefaultEncoding("UTF-8"); 
		// 파일당 업로드 크기 제한 (5MB) return multipartResolver; }
		multipartResolver.setMaxUploadSizePerFile(5 * 1024 * 1024); 
```

이는 application.yml로 보다 손 쉽게 처리할 수 있었습니다. 
``` yaml
	servlet:  
	  multipart:  
	    enabled: true  
	  max-file-size: 10MB  
	  max-request-size: 50MB
```

그리고 중요한건 어노테이션 매핑입니다.
```
@PostMapping("/upload")  
public ResponseEntity<Void> fhirBatch(@RequestPart("file") MultipartFile file) {  
    fhirFetcher.upload(file);  
    return ResponseEntity.ok().build();  
}
```

MultipartForm은 `@RequestPart`로 받아주어야 합니다. 

그리고 Postman으로 전송 시 헤더에 ContentType을 다음과 같이 넘겨줍니다. 
```
multipart/form-data; boundary=<calculated when request is sent>
```

전송 되는 파일 데이터는 boundary에 지정돼 있는 문자열로 구분한다고 합니다. 만약 해당 구문이 빠지면 파일 객체를 구별할 수 없게 되고 에러가 발생하게 됩니다.

[multipart/form 더 자세히 알아보기](https://lena-chamna.netlify.app/post/http_multipart_form-data/)
