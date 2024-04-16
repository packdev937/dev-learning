![[스크린샷 2024-04-16 오전 11.13.37.png]]

빈 생성을 하려고 했는데 다음과 같은 오류를 발견했습니다. 알고 보니 javax -> jarkarta로 변경하는 과정에서 흔히 볼 수 있는 문제였습니다.

기존
```bash
implementation 'com.querydsl:querydsl-jpa'  
implementation 'com.querydsl:querydsl-apt'
```

변경 후 
```bash
implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'  
annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"  
annotationProcessor 'jakarta.persistence:jakarta.persistence-api'  
annotationProcessor 'jakarta.annotation:jakarta.annotation-api'
```