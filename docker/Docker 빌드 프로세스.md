우선 아래 명렁어로 프로젝트를 .jar 파일로 빌드해줍니다. 
```
./gradlew build 
```

최상위 디렉토리에서 다음 명령어로 Dockerfile을 실행해줍니다.

```
docker build -t [username]/[repository 명] ./
```
- Tag를 붙인다면 `docker build -t packdev937/sunshine-server:v2 .`
- `docker build -t [username]/[repository name] ./` 명령어는 현재 디렉토리의 `Dockerfile`을 사용하여 Docker 이미지를 빌드하고, 이 이미지에 `[username]/[repository name]` 형식으로 태그를 지정합니다.
- 중요한건 `[username]/[repository 명]` 으로 하나의 이미지만 버전 업 하면서 올린다는 것입니다. 
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

### 트러블 슈팅 
exec /usr/openjdk-17/bin/java: exec format error 

알고보니 도커는 운영체제 플랫폼에 따라 빌드가 다르게 됩니다.
```
docker buildx create --use
docker buildx build --push --platform linux/amd64 -t [도커 사용자명]/[레포지토리 이름] .
```
ex) docker buildx build --push --platform linux/amd64 -t packdev937/sunshine-server .

**Is the docker demon running?**
![[스크린샷 2024-04-04 오후 1.52.54.png]]

도커를 실행해주어야 합니다.
``` bash
sudo systemctl start docker
```
