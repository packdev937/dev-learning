``` yml
services: 
	application: 
		image: sunshine 
		environment: 
		SPRING_DATASOURCE_URL: jdbc:mysql://[공개 IP 주소]:3306/sunshine?serverTimezone=Asia/Seoul 
		SPRING_DATASOURCE_USERNAME: root 
		SPRING_DATASOURCE_PASSWORD: 
	restart: always 
	container_name: sunshine 
	ports: - "8080:8080"

```