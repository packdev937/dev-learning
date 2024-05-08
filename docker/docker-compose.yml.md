### 1. Sample .yml 파일 
```
version: "3.3"

services:
  nginx:
    image: nginx:latest
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/log:/var/log/nginx
      - ./www:/var/www/html
    ports:
      - "80:80"

  certbot:
    depends_on:
      - nginx
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./certbot/etc:/etc/letsencrypt
      - ./certbot/var:/var/lib/letsencrypt
      - ./www:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email roypackgw@gmail.com --agree-tos --no-eff-email --force-renewal -d gdscsunshine.dev -d www.gdscsunshine.dev

  application:
    image: sunshine
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://34.64.63.248:3306/sunshine?serverTimezone=Asia/Seoul
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: Gdsc1234!
    restart: always
    container_name: sunshine
    ports:
      - "8080:8080"

```

완전 기본적인 docker-compose.yml은 다음과 같습니다.
```yml
version: "3"
  
services:
  app:
    container_name: server
    image : goodmoneying/server
    expose :
      - 8080
    ports :
      - 8080:8080
    restart : "always"
~                             
```

### 주의할 점
**docker-compose.yml**에 탭은 허용되지 않습니다. 공백으로 대체해야 합니다.