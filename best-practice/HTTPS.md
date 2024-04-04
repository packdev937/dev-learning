HTTPS를 적용하는 다양한 방법이 있지만 오늘 포스팅에셔는 docker-compose, nginx, certbot을 사용해서 HTTPS를 적용해보도록 하겠습니다. 그 과정에서 나오는 여러 용어들을 상세히 풀어볼 예정입니다. 
### HTTPS란
이해가 잘되는 예시가 있어 가져왔습니다. 
> 공인된 인증 기관(CA)에게 내 서버의 주민등록증(SSL 인증서) 을 발급 받고, 브라우저와 통신할 때 마다 주민등록증을 제시하며 자신이 인증됨을 알리는 방식입니다.

핵심은 (1) 인증 기관으로 부터 SSL 인증서를 발급 받고, (2) 브라우저가 내 서버에 접속할 때 마다 인증서를 제시하여 안전한 서버임을 입증하며, (3) 인증서에 적힌 내 서버의 **key** 를 통해 데이터를 암호화 하며 주고 받는 것입니다. 

##### 왜 HTTPS가 필요한가요?
네이버에 접속한다고 가정해보겠습니다. 
네이버에 로그인을 하기 위해서 아이디와 비밀번호를 입력합니다. 만약 해당 텍스트가 그대로 네이버가 아닌 악의적인 서버에 들어간다면 어떻게 될까요? 개인 정보가 유출되게 됩니다. 
따라서 (1) 우리는 뒤죽박죽 암호화가 된 텍스트를 서버로 전송합니다. 
(2) 또한, 접속한 사이트가 진품인지 가품인지 판별해주는 역할을 합니다.

즉, 기관으로부터 검증된 사이트만 HTTPS가 허용되기 때문에 HTTPS를 사용한다는 것은 안전하다는 반증이겠죠!

##### 암호화된 텍스트를 어떻게 복호화하나요?
여기서는 비대칭키를 알아야 합니다. 네이버 서버는 개인키와 공개키를 갖고 있습니다. 그리고 오직 공개키만 공개하죠. 사용자는 데이터를 공개키로 암호화 하여 보냅니다. 그리고 암호화된 데이터는 오직 개인키로만 복호화 가능합니다. 

##### 사용자와 서버가 통신하는 과정 
1) 클라이언트는 어떤 랜덤 데이터를 생성해서 서버로 보냅니다
2) 해당 데이터를 받은 서버는 생성한 무작위 데이터 값과 인증서를 함께 클라이언트에게 보냅니다
3) 이를 handshake라고 합니다
4) 클라이언트는 해당 인증서가 진짜인지 브라우저에 내장된 CA 들의 정보로 확인합니다
5) CA의 인증을 받은 인증서들은 해당 CA의 개인키로 암호화 돼있습니다
6) 즉, 브라우저의 내장된 CA의 개인 키로 복호화가 가능한 인증서라면 안전한 서버라는 것을 입증할 수 있습니다 
7) 성공적으로 복호화 된 인증서에는 서버의 공개키가 있습니다. 
8) 하지만 비대칭 키를 사용하여 통신을 하는 것이 아닙니다. 서버에서 개인 키로 복호화 하는 것은 많은 비용이 들기 때문입니다
9) 해당 비대칭 키는 클라이언트와 서버 간 대칭 키를 공유할 때 사용됩니다. 대칭 키가 비밀로 유지가 된다면 해당 대칭 키로 통신을 이어나가도 보안적으로 전혀 문제가 없기 때문입니다. 

### 가비아에서 도메인 구입 
도메인을 EC2에 적용하는 방법은 다음과 같습니다. 
- EC2 에 탄력적 IP (고정 IP) 적용
- EC2의 IP를 내 도메인의 A 타입과 매핑 

🤔 도메인에서 A, AAA, CNAME 등이 뭘 의미할까요?
서브 도메인을 등록하기 위해서는 A, CNAME 같은 레코드 정보를 등록해야 합니다. 이러한 레코드 정보가 왜 필요하고 어떻게 사용될까요? 

#### [DNS](https://dev.plusblog.co.kr/30#google_vignette)
인터넷을 구성하고 있는 IP 주소는 127.0.0.1 과 같이 숫자로 구성됩니다. 이러한 숫자는 사람이 일일이 외우기 힘들기 때문에 문자열로 IP 주소를 표현하기 시작하였습니다. 다만 실제 통신에서는 www.naver.com 을 접속하면 IPv4 주소로 변환해 주는 작업이 필요합니다. 이런 서비스를 DNS 서비스 라고 합니다. 

DNS 서버는 도메인 주소와 IP 주소가 쌍으로 테이블에 저장되어 있습니다. 
레코드는 해당 테이블에서 하나의 행을 의미합니다. 

**A 레코드**는 DNS에 저장되는 정보의 타입으로 도메인 주소와 서버의 IP 주소가 직접 매핑시키는 방법입니다. 테이블에서 naver.com 같은 것이 도메인 주소, 192.168.0.1 같이 숫자로 이루어진 주소가 IP 주소입니다.

**CNAME 레코드**는 Canonical Name의 약자로 도메인 주소를 또 다른 도메인 주소로 매핑하는 것을 의미합니다. 
예를 들어, dev.naver.com | www.naver.com 이 있고 www.naver.com | 192.168.0.1 이 있다고 가정해봅시다. dev.naver.com 으로 접속했다면 DNS 서버는 www.naver.com을 반환하고 다시 www.naver.com을 DNS 서버에 요청해 IP 주소를 얻어내게 됩니다. 

A 레코드와 CNAME 레코드의 장단점은 다음과 같습니다.

A 레코드의 장점은 한 번의 요청으로 찾아갈 서버의 IP 주소에 바로 접근 가능하다는 점입니다. 단점은 여러 도메인 주소가 하나의 IP 주소를 참조하고 있을 때, 해당 주소가 바뀐다면 모든 A 레코드를 변경해주어야 합니다.

CNAME 레코드는 여러 번의 요청이 필요하지만 반대로 주소가 바뀐다면 참조하고 있는 도메인 주소의 IP 주소만 변경해주면 됩니다. 

### [Nginx 설정](https://dkswnkk.tistory.com/675)
EC2에 직접 설정할 수도 있고 도커로 Nginx 컨테이너를 설치할 수 있습니다. 우선 통상적인 Nginx 설정 방법에 대해 알아보고자 합니다. 

Nginx의 설정 파일을 수정해주어야 하는데 대게 설정 파일은 /etc/nginx 밑에 위치하고 있습니다. 

만약 설정 파일을 못 찾겠다면 다음을 설정해주면 됩니다. 
```bash
find / -name nginx.conf
```

### docker란? 
도커(Docker)는 애플리케이션을 컨테이너(Container)라는 격리된 환경에서 실행할 수 있도록 해주는 오픈 소스 플랫폼입니다. 컨테이너는 가볍고 휴대가능하며, 운영체제 수준에서 격리되어, 애플리케이션이 실행되는 환경을 코드와 함께 패키징하여 어느 환경에서나 동일하게 작동하도록 보장합니다.
##### 도커의 주요 개념

- **이미지(Image)**: 애플리케이션과 그 실행에 필요한 모든 파일, 라이브러리, 의존성을 포함하는 경량의, 실행 가능한 소프트웨어 패키지입니다. 이미지는 애플리케이션이 실행되기 전의 상태를 나타냅니다.
- **컨테이너(Container)**: 도커 이미지를 기반으로 실행된 인스턴스입니다. 컨테이너는 도커 이미지를 실행할 때 생성되며, 애플리케이션이 실제로 작동하는 격리된 환경입니다.
- **도커 파일(Dockerfile)**: 도커 이미지를 생성하기 위한 명령어와 설정이 포함된 텍스트 파일입니다. 이 파일은 이미지 빌드 과정을 자동화합니다.
##### 도커의 장점
- **환경 일관성**: 개발, 테스트, 프로덕션 환경 간의 차이를 최소화하여, 소프트웨어를 더 빠르고 안정적으로 배포할 수 있습니다.
- **휴대성**: 어떤 환경에서도 동일하게 작동하는 컨테이너를 생성할 수 있으므로, 애플리케이션을 쉽게 이전, 배포 및 확장할 수 있습니다.
- **가볍고 빠름**: 가상머신에 비해 컨테이너는 더 가볍고 빠르며, 리소스 사용이 효율적입니다. 이는 컨테이너가 게스트 운영체제를 필요로 하지 않기 때문입니다.
- **격리성**: 각 컨테이너는 서로 독립적으로 실행되므로, 한 애플리케이션의 변경이나 오류가 다른 애플리케이션에 영향을 주지 않습니다.
- **자동화**: 도커의 자동화 도구는 개발 및 배포 과정을 간소화하고, CI/CD(지속적 통합/지속적 배포) 파이프라인과의 통합을 용이하게 합니다.
##### EC2에 docker 설치 
[[AWS EC2 인스턴스에 Docker 설치하기]]

### docker-compose 세팅 
우선 nginx, certbot 모두 Docker Library에서 지원합니다. 

`docker-compose.yml`를 작성하기 전 조금 더 자세히 알아봅시다. 
- 서비스 이름 
	services: 
		server:
	위와 같이 설정 파일이 구성되어 있을 때 에서 server가 서비스 이름입니다. 
	서비스 이름은 네트워크 설정이나 Docker Compose의 다른 부분에서 서비스를 참조할 때 사용될 수 있습니다. 
	
	서비스의 이름은 유일 (unique) 해야되며 명확하고 기억하기 쉬워야 합니다. 
- image는 서비스에서 사용할 컨테이너의 이미지를 지정합니다. 
	이미지의 이름은 "이미지 이름 : 태그" 형식으로 작성합니다.
	image : nginx:1.19.0-alpine
	이미지는 크게 공식 이미지와 사용자 정의 이미지로 나뉠 수 있습니다. 
	
	공식 이미지는 특별한 이름 공간에서 관리 됩니다. `eg) 'nginx'`
	
	사용자 생성 이미지를 사용할 수도 있습니다.
	형식은 `[username]/[image 이름] : [tag]` 입니다.
- ports
	"호스트 포트 : 컨테이너 포트" 로 작성됩니다.
	ports:
		- "5000:5000"
	호스트의 5000 포트를 컨테이너의 5000 포트에 연결한다는 의미입니다. 
	
	호스트의 포트와 컨테이너 포트는 어떤 차이가 있을까요?
	호스트 포트를 통해서 외부 네트워크에서 호스트 시스템에 접근할 수 있습니다. 호스트 포트를 통해 들어온 트래픽은 지정된 컨테이너 포트로 전달됩니다. 
	
	컨테이너 포트는 컨테이너 내부의 네트워크 포트입니다. 컨테이너 내부에서 실행 중인 애플리케이션은 해당 포트를 통해 네트워크 트래픽을 수신합니다. 
	
	이를 통해 여러 컨테이너가 같은 포트를 사용하고 있을 때 각 컨테이너를 다른 호스트 포트에 매핑함으로써 포트 충돌을 방지할 수 있습니다. 
	
	예를 들어, 스프링 부트에서 포트 번호를 8081로 설정하고 호스트 포트는 8080으로 설정했다면 호스트 시스템의 8080 포트로 보내진 요청이 8081로 전달됩니다. 
- container_name
	컨테이너를 

볼륨 마운트를 설정하는부분입니다.

- volumes : 호스트와 컨테이너 사이의 볼륨을 설정합니다. 볼륨은 데이터를 컨테이너가 아닌 호스트 시스템에 저장하게 해서 컨테이너가 삭제되거나 재시작될 때도 데이터를 유지하도록 하는 설정입니다. 

	예를 들어, `./data/nginx:/etc/nginx/conf.d` 다음과 같은 설정을 했다고 가정해봅시다. 이는 /data/nginx 디렉토리와 Nginx 컨테이너 내부의 /etc/nginx/conf.d 를 연결하는 것입니다.
	

```yaml
version: '3'
services:  
  server:
    container_name : server
    image : goodmoneying/server
    ports :
      - 8080:8080
    restart: "always"

  nginx:  
    image: nginx:1.15-alpine  
    ports:  
      - 80:80
      - 443:443 
    volumes:  
      - ./data/nginx:/etc/nginx/conf.d 
      - ./data/certbot/conf:/etc/letsencrypt  
	  - ./data/certbot/www:/var/www/certbot
	depends_on :
	  - "server"

  certbot:  
    image: certbot/certbot
    volumes:  
	  - ./data/certbot/conf:/etc/letsencrypt  
	  - ./data/certbot/www:/var/www/certbot
```

여기서 example.org는 우리의 도메인으로 바꿔줘야 합니다. 
- 참고로 해당 파일의 경로는 다음과 같습니다. (`data/nginx/app.conf`)
```
server {  
    listen 80;  
    listen [::]:80;
    server_name example.org;    
    
    location / {  
        return 301 https://$host$request_uri;
    }

	location /.well-known/acme-challenge/ {  
		root /var/www/certbot;  
	}	  
}

server {  
    listen 443 ssl;  
    server_name example.org;  

	ssl_certificate /etc/letsencrypt/live/packdev937.site/fullchain.pem;  
	ssl_certificate_key /etc/letsencrypt/live/packdev937.site/privkey.pem;
	include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 

	location / {
        proxy_pass  http://server:8080;
        proxy_set_header    Host                $http_host;
        proxy_set_header    X-Real-IP           $remote_addr;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    } 
}
```

init-letsencrypt.sh 도 홈 디렉토리 경로에 넣어줍니다. 
```
curl -L "https://raw.githubusercontent.com/wmnnd/nginx-certbot/master/init-letsencrypt.sh" > init-letsencrypt.sh

chmod +x init-letsencrypt.sh
```

```
#!/bin/bash

if ! [ -x "$(command -v docker-compose)" ]; then
  echo 'Error: docker-compose is not installed.' >&2
  exit 1
fi

domains=(goodmoneying.shop)
rsa_key_size=4096
data_path="./data/certbot"
email="good.moneying.2024@gmail.com" # Adding a valid address is strongly recommended
staging=0 # Set to 1 if you're testing your setup to avoid hitting request limits

if [ -d "$data_path" ]; then
  read -p "Existing data found for $domains. Continue and replace existing certificate? (y/N) " decision
  if [ "$decision" != "Y" ] && [ "$decision" != "y" ]; then
    exit
  fi
fi


if [ ! -e "$data_path/conf/options-ssl-nginx.conf" ] || [ ! -e "$data_path/conf/ssl-dhparams.pem" ]; then
  echo "### Downloading recommended TLS parameters ..."
  mkdir -p "$data_path/conf"
  curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf > "$data_path/conf/options-ssl-nginx.conf"
  curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot/certbot/ssl-dhparams.pem > "$data_path/conf/ssl-dhparams.pem"
  echo
fi

echo "### Creating dummy certificate for $domains ..."
path="/etc/letsencrypt/live/$domains"
mkdir -p "$data_path/conf/live/$domains"
docker-compose run --rm --entrypoint "\
  openssl req -x509 -nodes -newkey rsa:$rsa_key_size -days 1\
    -keyout '$path/privkey.pem' \
    -out '$path/fullchain.pem' \
    -subj '/CN=localhost'" certbot
echo


echo "### Starting nginx ..."
docker-compose up --force-recreate -d nginx
echo

echo "### Deleting dummy certificate for $domains ..."
docker-compose run --rm --entrypoint "\
  rm -Rf /etc/letsencrypt/live/$domains && \
  rm -Rf /etc/letsencrypt/archive/$domains && \
  rm -Rf /etc/letsencrypt/renewal/$domains.conf" certbot
echo


echo "### Requesting Let's Encrypt certificate for $domains ..."
#Join $domains to -d args
domain_args=""
for domain in "${domains[@]}"; do
  domain_args="$domain_args -d $domain"
done

# Select appropriate email arg
case "$email" in
  "") email_arg="--register-unsafely-without-email" ;;
  *) email_arg="--email $email" ;;
esac

# Enable staging mode if needed
if [ $staging != "0" ]; then staging_arg="--staging"; fi

docker-compose run --rm --entrypoint "\
  certbot certonly --webroot -w /var/www/certbot \
    $staging_arg \
    $email_arg \
    $domain_args \
    --rsa-key-size $rsa_key_size \
    --agree-tos \
    --force-renewal" certbot
echo

echo "### Reloading nginx ..."
docker-compose exec nginx nginx -s reload
```

실행은 다음과 같이 진행합니다.
```
sudo ./init-letsencrypt.sh
docker-compose up
```

### 문제 발생 
- 리다이렉션 요청이 너무 많습니다. 
	data/nginx/app.conf 에 proxy_pass가 동일한 도메인으로 되어 있지는 않은지 확인하세요.
	이 때 http://[컨테이너_이름]:8080 으로 진행해야 합니다. 
- 
### Reference 
[1](https://velog.io/@lamazzang25/Docker-compose-certbot-nginx-%EB%A1%9C-ssl-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B0%9C%EA%B8%89%ED%95%98%EA%B8%B0)
[2](https://pentacent.medium.com/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71)
[3](https://velog.io/@crab4862/AWS-%ED%95%98%EB%82%98%EC%9D%98-%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9C%BC%EB%A1%9C-%EB%91%90-%EA%B0%9C%EC%9D%98-%EC%9B%B9%EC%82%AC%EC%9D%B4%ED%8A%B8-%ED%98%B8%EC%8A%A4%ED%8C%85%ED%95%98%EA%B8%B0-SubDomain)

