Consumer는 특정 토픽의 파티션에서 레코드를 조회하는 역할을 합니다. 
```java
Properties prop = new Properties();
prop.put("bootstrap.servers", "localhost:9092");
prop.put("group.id", "group1");
prop.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer <String, String> consumer = new KafkaConsumer<String, String> (prop);

consumer.subscribe(Collections.singleton("simple"));
while(조건) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
	for (ConsumerRecord<String, String> record : records){
		System.out.println(record.value() + ":" + record.topic() + ":" + record.partition() + ":" + record.offset());
	}
}

consumer.close();
```

