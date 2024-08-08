따로 설정할 부분이 없다면 S3 -> 버킷 생성을 누르고 퍼블릭 엑세스 차단 설정만 풀어 준 뒤 그대로 생성해줍니다.

버킷이 만들어진 모습은 다음과 같습니다.
![[스크린샷 2024-08-04 오후 6.56.11.png]]

그리고 IAM 사용자를 만들어줍니다. 
(IAM > 엑세스 관리 > 사용자 > 사용자 추가)

직접 정책을 연결해주어야 합니다. (직접 정책 연결 선택)
- AmazonS3FullAccess

IAM 사용자 생성 후 엑세스 키를 생성합니다. 
(IAM > 엑세스 관리자 > 사용자 > 생성한 사용자 이름 클릭 > 보안 자격 증명 > 엑세스 키)

`application.yml`
```yaml
cloud:  
  aws:  
    s3:  
      bucket: <버킷 이름>
    stack.auto: false  
    region.static: ap-northeast-2  
    credentials:  
      accessKey: <발급 받은 accessKey>  
      secretKey: <발급 받은 secretKey>
```
## Reference
- https://gaeggu.tistory.com/33