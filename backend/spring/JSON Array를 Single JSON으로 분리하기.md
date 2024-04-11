여러 JSON이 묶여 있는 FHIR 데이터를 분리할 필요가 있습니다. 

제가 원하는 방식을 쉬운 예제로 설명하면 다음과 같습니다.
```json
{
	entry : [ 
		{ "record": "a", "actions": [ { "id": 1 }, { "id": 2 } ] }, 
		{ "record": "b", "actions": [ { "id": 3 }, { "id": 4 } ] } 
	]
}
```

분리 후 데이터 예시 
```json
{record: "a", "action": { "id": 1 }},
{record: "a", "action": { "id": 2 }},
{record: "b", "action": { "id": 3 }},
{record: "b", "action": { "id": 4 }}
```

우선 Jackson 라이브러리가 아닌 별도의 JSON 라이브러리를 다운 받아 사용해보고자 합니다. 
`build.gradle`
```java
implementation 'org.json:json:20240303'
```

추가적인 정보는 [다음](https://velog.io/@chosj1526/Java-JSON-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95-JSONObject-JSONArray-JsonParser%EB%A1%9C-%ED%8C%8C%EC%8B%B1%ED%95%98%EA%B8%B0)을 확인합니다. 
