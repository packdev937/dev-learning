## 1. 개요
Elastic Search는 다음 웹사이트로 이동하여 다운 받아 줍니다. 
https://www.elastic.co/kr/downloads/elasticsearch

하지만 맥북의 보안 정책 상 외부에서 다운로드 받은 파일의 경우 "Apple에서 악성 소프트웨어가 있는지 확인 수 없기 때문에 열 수 없습니다." 오류가 발생했습니다. 

이는 다음과 같이 해결할 수 있습니다. 
```
// 게이트키퍼 끄기
sudo spctl --master-disable

// 게이트키퍼 켜기
sudo spctl --master-enable
```