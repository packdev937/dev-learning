## 1. 개요
Elastic Search를 초기에 설치하는 방법은 꽤나 까다로운 것 같습니다. 따라서 설치 과정을 기록해가며 진행해볼 예정입니다. 
## 2. Elastic Search 설치 후 실행 
https://www.elastic.co/kr/downloads/elasticsearch

``` bash
> ./elasticsearch
```

다음과 같은 오류가 발생하였습니다. 
![[스크린샷 2024-05-31 오전 11.03.24.png]]

디스크 용량이 부족해서 다음과 같은 오류가 날 수 있다는 정보를 찾은 뒤, 용량을 비우고 시도하니 오류 메세지는 더이상 출력되지 않았습니다. 

이제 정상적으로 실행이 됐나 확인하려고 https://localhost:9200 으로 접속해보았습니다. 그러니 다음과 같은 로그인 창이 떴습니다. 

![[스크린샷 2024-05-31 오전 11.10.48.png|400]]

저는 어떠한 아이디도 설정한 적이 없습니다. 찾아보니 슈퍼 계정의 아이디는 `elastic`이며 비밀번호는 별도의 초기화 과정을 거쳐야 한다고 합니다. 따라서 다음 명령어를 통해 비밀번호를 초기화해주었습니다.
``` bash
bin/elasticsearch-reset-password -u elastic
```

![[스크린샷 2024-05-31 오전 11.18.05.png]]

로그인 성공 시 다음과 같은 JSON 응답을 받을 수 있습니다. 
``` json
{
  "name" : "Gundorit-MacBook.local",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "zQxSCc3GS0Wk-lho8G0z0w",
  "version" : {
    "number" : "8.13.4",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "da95df118650b55a500dcc181889ac35c6d8da7c",
    "build_date" : "2024-05-06T22:04:45.107454559Z",
    "build_snapshot" : false,
    "lucene_version" : "9.10.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 3. Kibana 설치 후 실행
갑자기 뜬금없이 Elastic Search 설치 후 Kibana를 설치해야될까요? 

Elasticsearch와 Kibana는 상호 보완적인 역할을 합니다. 흔히 말하는 ELK의 E와 K가 Elastic Search와 Kibana인 것입니다. 

Kibana 설치는 다음 링크에서 진행합니다. 
[Here the link](https://www.elastic.co/kr/downloads/kibana)

`/bin/kibana.bat`을 실행시켜줍니다. http://localhost:5601 로 접속해서 실행을 확인할 수 있습니다. 

+참고로 Enrollment Token을 발급받아야 하는데 이는 상세히 설명해줍니다. 
`./bin/elasticsearch-create-enrollment-token --scope kibana`

안내에 따라 로그인 까지 마치면 다음과 같은 콘솔창이 뜨게 됩니다.
![[스크린샷 2024-05-31 오후 12.49.02.png]]
- https://dnai-deny.tistory.com/71