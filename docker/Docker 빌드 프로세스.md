우선 아래 명렁어로 프로젝트를 .jar 파일로 빌드해줍니다. 
```
./gradlew build 
```

최상위 디렉토리에서 다음 명령어로 Dockerfile을 실행해줍니다.

```
docker build -t [username]/[repository 명] ./
```
- Tag를 붙인다면 `docker build -t packdev937/sunshine-server:v2 .`

docker images로 이미지가 생성된걸 확인할 수 있습니다.
```
docker images
```

그리고 이미지를 docker hub에 push 해줍니다.
```
docker push packdev937/sunshine-server
```

후에 원격 서버에서 다시 docker login을 진행합니다.
```
docker login
```

docker hub에서 이미지를 가져옵니다.
```
sudo docker pull packdev937/sunshine-server
```

이미지를 `docker-compose.yml`에 설정한 대로 바꿔줍니다. 이는 `docker-compose`가 설정된 태그를 가진 이미지를 찾고 사용하기 때문입니다.
```
sudo docker tag packdev937/sunshine-server sunshine
```

실제 사용된[[docker-compose.yml]] 예시

백 그라운드에서 실행될 수 있도록 nohup 명령어를 함께 사용합니다.
```
sudo nohup docker-compose up &
```



### **만약 이미 배포된 이미지가 있다면** 
sudo docker-compose down
sudo docker rmi [삭제할 이미지]
sudo docker pull packdev937/sunshine-server:[tag]
sudo docker tag packdev937/sunshine-server sunshine
+ 뒤에 sunshine은 docker-compose.yml에 등록한 이름입니다.
// 따로 태그를 안달아주면 최신 버전으로 업데이트 되는 듯 합니다. 


+ sudo docker-compose logs -f 로 로그를 확인


### exec /usr/openjdk-17/bin/java: exec format error 
알고보니 도커는 운영체제 플랫폼에 따라 빌드가 다르게 된다. 
```
docker buildx create --use
docker buildx build --push --platform linux/amd64 -t [도커 사용자명]/[레포지토리 이름] .
```
ex) docker buildx build --push --platform linux/amd64 -t packdev937/sunshine-server .

(매번 이걸로 빌드하자)
### 주의할 점
터미널에서 실행하자 