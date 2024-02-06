
docker build -t packdev937/sunshine-server ./
- Tag를 붙인다면 `docker build -t packdev937/sunshine-server:v2 .`
docker push packdev937/sunshine-server

후에 원격 서버에서 
docker login
sudo docker pull packdev937/sunshine-server
sudo docker tag packdev937/sunshine-server sunshine
sudo nohup docker-compose up &


### **만약 이미 배포된 이미지가 있다면** 
sudo docker-compose down
sudo docker rmi [삭제할 이미지]
sudo docker pull packdev937/sunshine-server:[tag]
sudo docker tag packdev937/sunshine-server:v2 **sunshine** 
+ 뒤에 sunshine은 docker-compose.yml에 등록한 이름입니다.
// 따로 태그를 안달아주면 최신 버전으로 업데이트 되는 듯 합니다. 


+ `docker-compose logs -f` 로 로그를 확인


### exec /usr/openjdk-17/bin/java: exec format error 
알고보니 도커는 운영체제 플랫폼에 따라 빌드가 다르게 된다. 
```
docker buildx create --use
docker buildx build --push --platform linux/amd64 -t [도커 사용자명]/[레포지토리 이름] .
```