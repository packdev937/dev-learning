우선 Docker Image 배포가 어떻게 진행되었는지 확인할 필요가 있습니다. 

- Dockerfile 만들기
- docker build --tag [Repository Name]/spring-app
- docker push [Repository Name]/spring-app

-> 여기 까지는 진행 된 상태라고 가정합니다

docker login 하는 과정이 필요합니다.
```
docker login 
```

Docker Hub에서 이미지를 받아옵니다. 

`ci.yml`
```
name : CI

on:
  pull_request:
    branches: [dev]
    
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses : actions/checkout@v2
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
         java-version: '17'
         distribution: 'temurin'

      - name: permission for gradlew
        run: chmod +x ./gradlew

      - name: test with gradle
        run: ./gradlew test
        shell: bash
```

deploy.yml
```
name: Build and Deploy Spring Boot to AWS EC2

on:
  pull_request:
    branches: [dev]

env:
  PROJECT_NAME: EatSSU-Server
  BUCKET_NAME: eatssu-bucket
  CODE_DEPLOY_APP_NAME: github-actions-eatssu
  DEPLOYMENT_GROUP_NAME: github-actions-eatssu-group

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: '17'
          distribution: 'temurin'

// 여기서 application.yml도 추가되야 할듯 
      - name: Permission for gradle
        run: chmod +x ./gradlew

      - name: Build with gradle
        run: ./gradlew build

      - name: Make Zip File
        run: zip -qq -r ./$GITHUB_SHA.zip .

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: Upload to S3
        run: aws s3 cp $GITHUB_SHA.zip s3://$BUCKET_NAME/$PROJECT_NAME/$GITHUB_SHA.zip

      - name: CodeDeploy
        run: aws deploy create-deployment --application-name $CODE_DEPLOY_APP_NAME --s3-location bucket=$BUCKET_NAME,key=$PROJECT_NAME/$GITHUB_SHA.zip,bundleType=zip --deployment-group-name $DEPLOYMENT_GROUP_NAME --region ap-northeast-2

```

appspec.yml
```
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ec2-user/EatSSU-Server

permissions:
  - object: /home/ec2-user/EatSSU-Server
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: /home/ec2-user/EatSSU-Server/scripts/deploy.sh
      timeout: 60
      runas: ec2-user
```

deploy.sh
```
#!/bin/bash

# 환경 설정
REPOSITORY=/home/ec2-user/EatSSU-Server
DOCKER_USERNAME=your_dockerhub_username 
DOCKER_PASSWORD=your_dockerhub_password

LOG_DIR=$REPOSITORY/logs
LOG_FILE="$LOG_DIR/$(date +'%Y-%m-%d').log"
VERSION_FILE="$REPOSITORY/version.txt"
APP_NAME=eatssu-server # 소문자로 작성

# 로그 디렉토리 확인 및 생성
if [ ! -d "$LOG_DIR" ]; then
  sudo mkdir -p "$LOG_DIR"
  sudo chown ec2-user:ec2-user "$LOG_DIR"
fi

# 버전 번호 업데이트
if [ ! -f "$VERSION_FILE" ]; then
  echo "0" > "$VERSION_FILE"
fi

VERSION=$(cat "$VERSION_FILE")
NEW_VERSION=$((VERSION+1))
echo "$NEW_VERSION" > "$VERSION_FILE"

# 기존 자바 애플리케이션 프로세스 종료
cd $REPOSITORY
CURRENT_PID=$(pgrep -f $APP_NAME)
if [ -z "$CURRENT_PID" ]; then
  echo "> 종료할 것 없음."
else
  echo "> kill -15 $CURRENT_PID"
  sudo kill -15 $CURRENT_PID
  sleep 5
fi

# Docker 관련 명령어 실행
docker login --username = $DOCKER_USERNAME --password = $DOCKER_PASSWORD

# Docker Compose를 이용해 기존 컨테이너 종료
sudo docker-compose down

# Docker 이미지 빌드 및 푸시
sudo docker build --tag soy2on/eatssu-dev-spring:$NEW_VERSION .
sudo docker build --tag soy2on/eatssu-dev-spring:latest .
sudo docker push soy2on/eatssu-dev-spring:$NEW_VERSION
sudo docker push soy2on/eatssu-dev-spring:latest

# 도커 컴포즈를 이용해 컨테이너 실행
cd ..
nohup sudo docker-compose up &

# 로그 파일로 리디렉션
echo "> $APP_NAME 배포" > $LOG_FILE 2>&1

```


## Reference 
- https://etloveguitar.tistory.com/71
- https://mumomu.tistory.com/132