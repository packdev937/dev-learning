
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