## CI/CD 란? 
> CI/CD (Continuous Integration/Continuous Delivery)

#### CI
CI 는 **Continuous Integration** 로 지속적인 통합을 의미합니다. 
개발 간 기능 추가, 버그 수정, 리팩토링과 같은 코드 변경을 주기적으로 통합하는 것을 의미합니다 .

변경은 하루에도 몇 번씩 빈번하게 발생하며 이 때 통합되는 코드의 양은 작아야 합니다. 추가적으로 수시로 일어나는 통합 코드가 문제 없음을 확인 하기 위해서는 테스트 코드가 필수적으로 작성되어야 하며, 이 과정 또한 자동화가 되어야 합니다. 

#### CD 
CD는 **Continuous Delivery**  혹은 **Deployment를** 뜻합니다. 지속적인 서비스 제공 및 배포를 의미합니다.

앞서 본 CI 단계에서는 코드를 빌드하고 테스트를 진행합니다. 테스트가 정상적으로 통과했다면 이제는 배포를 할 차례입니다. 이를 CD 단계에서 진행합니다.

Continuous Delivery 는 Repository로 Release 하는 것을 뜻하고, Deployment는 Production 레벨 까지 자동화된 배포를 의미합니다.

#### CI/CD Tools
CI/CD 는 다양하게 구현될 수 있습니다. 
이번 포스팅에서는 Github Actions와 AWS를 활용해서 진행해보고자 합니다. 

### Github Actions
#### Repository 생성
	간단한 실습용 Repository를 생성합니다.
![[CI-CD test(1).png]]

그리고 테스트 여부를 판단하기 위한 간단한 테스트 코드도 작성합니다.
![[CI-CD test(2).png]]

다음으로 `.github/workflows/ci.yml` 파일에 CI 워크플로우를 정의해야 합니다. 여기에는 일반적으로 프로젝트에 필요한 다양한 단계가 포함됩니다:
![[CI-CD test (3).png]]
![[CI-CD test (4).png]]
##### name

```yml
name : CI
```

먼저 `name` 필드를 작성해줍니다. 여기에는 어떤 이름이든지 올 수 있습니다. 
해당 이름은 워크 플로우의 식별자 역할을 합니다.

##### on 

```
on: push
```

다음으로 on 입니다. 워크플로를 실행할 수 있는 이벤트를 정의하는 데 사용됩니다.
예를 들어, 위와 같이 설정된다면 branch에 푸쉬가 발생하면 다음 값을 가진 워크 플로가 실행됩니다. 

```
on: [push, pull_request]
```
여러 이벤트도 사용 가능합니다. 다음과 같이 설정되면 push, pr 시 워크 플로가 실행됩니다.

추가적인 응용 버전을 작성해보겠습니다. 
```
on: 
	push:
		branches:
			[main, dev]
```

위 설정은 main 또는 dev 브랜치에 push 되었을 때만 워크 플로가 작동합니다. 

더 다양한 정보는 다음을 확인합니다.
- https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions

##### jobs
```
jobs: 
	build: 
		runs-on: ubuntu-latest
```
runs-on는 작업이 실행될 가상 환경을 지정합니다. ubuntu-latest는 가장 최신 버전의 Ubuntu 운영체제를 선택합니다. 

다양한 이유가 있지만 높은 호환성을 꼽을 수 있으며, 많은 Github Actions을 사용하는 예제들이 ubuntu-latest를 사용합니다. 
(특정 요구 사항에 따라 'windows-latest', 'macos-latest' 들이 사용될 수 있습니다)

##### 🤔 가상 환경 무슨 소리지?
GitHub Actions의 가상 환경은 GitHub의 서버에 구축된, 사전 구성된 운영 체제입니다. 각 가상 환경에는 개발에 필요한 다양한 도구 (git, docker, java, python) 등이 포함될 수 있습니다. 

다시 말해, Github Actions의 가상 환경에서 다양한 개발 도구들을 이용해 자동으로 빌드와 테스트를 진행한 뒤 CD한테 작업을 넘긴다고 생각할 수 있습니다.
##### steps
```yml
steps:
	- name: Checkout 
	  uses: actions/checkout@v2
	- name: Set up JDK 17
	  uses: actions/setup-java@v2
	  with:
		  java-version: '17'
		  distribution: 'temurin'
	- name: permission for gradle
	  run: chmod +x ./gradlew
	- name: build gradle
	  run: ./gradlew build
```

actions/checkout 액션은 사용자의 리포지토리 코드를 Checkout 하여 러너에 복제함으로써, 워크플로우에서 빌드, 테스트, 배포 등의 작업을 수행할 수 있도록 합니다.

과정은 다음과 같습니다. 
- 워크 플로우가 실행되고 러너가 할당됨 
- `actions/checkout`을 통해 Github 마켓 플레이스에서 러너로 코드를 가져옴 
- 실행 중인 러너의 파일 시스템에 사용자의 repository를 복제함 
- 러너에 Checkout 된 코드를 사용하여 워크 플로우에 정의한 나머지 작업들을 실행

참고로 actions/checkout은 @v2, @v3 등 버전을 설정할 수 있습니다.
##### 러너가 뭔데?
GitHub Actions 러너는 GitHub에서 제공하는 서버입니다. 이 서버에서는 사용자가 정의한 워크플로우에 따라 여러 작업이 실행됩니다. 따라서 actions/checkout을 사용한다는 건 러너의 파일 시스템에 Github repository 코드가 복제된다는 뜻입니다. 

그 외에 코드들은 자바 환경을 설정하고, 빌드하는 명령어들을 일련의 순서로 배치한 것입니다. 

참고로 테스트와 빌드를 분리할 수 있습니다.
```
name : CI

on : 
  push:
    branches: [main]

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

위와 같이 작성한다면 Github Actions에서 진행하고 빌드는 AWS EC2에서 진행할 수 있습니다. 
## AWS 
Github Actions 이후의 과정은 다음과 같습니다. 
![[CI-CD flow.png]]
#### IAM 생성
우선 S3, EC2 사용에 앞서 IAM을 생성합니다. 그 이유는 IAM을 사용하여 리소스에 대한 다양한 권한을 설정할 수 있기 때문입니다. 위 설정을 통해 루트 사용자가 아닌 다른 사용자도 사용자가 허락한 수준에서의 권한을 사용할 수 있습니다. 

따라서 IAM 사용자 또는 역할 생성하여 엑세스 키와 비밀 키를 발급 받고, 이를 Github Secrets에 등록합니다. 그리고 해당 키를 사용하여 S3 버킷에 파일을 업로드 하는 등의 작업을 수행합니다. 

![[CD-CD IAM (1).png]]
우선 IAM > 사용자 > 사용자 생성으로 가줍니다. 

S3FullAccess와 CodeDeployFullAccess 정책을 추가해줍니다.
![[CI-CD IAM (2).png]]

이제 엑세스 키를 만들어줍니다.
![[CD-CD IAM (3).png]]
![[CD-CD IAM (4).png]]
![[CD-CD IAM (5).png]]
해당 키는 Github Actions을 적용하고자 하는 Repository에 등록합니다.

Access Key : `AKIAR6JYVZPLDCMAMOQH`
Secret Access Key : `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`

![[CD-CD IAM (6).png]]

![[CD-CD IAM (7).png]]

다음은 EC2, CodeDeploy 에 필요한 역할을 생성할 차례입니다.
![[CD-CD IAM (8).png]]

![[CD-CD IAM (9).png]]

우선 EC2에 쓰일 역할에는 AWSS3FullAccess와 AWSCodeDeployFullAccess를 추가해줍니다.

동일하게 CodeDeploy에 쓰일 역할을 만들어 줍니다. 이 때, 권한은 AWSCodeDeployRole 입니다.

### S3 
다음으로 S3 버킷을 생성해줄 차례입니다. S3는 jar 파일을 담기 위해 필요한 인스턴스 입니다. 

![[CD-CD S3 (1).png]]

### EC2
먼저 EC2 인스턴스를 생성하고, EC2에 역할 수정을 눌러 이전에 만들었던 역할로 변경합니다. 
![[CD-CD EC2 (1).png]]

다음 EC2에 필요한 프로그램들을 접속하여 설치해 줍니다. 
```
sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-17-x64-linux-jdk.rpm -o jdk17.rpm

sudo yum localinstall jdk17.rpm
java --version

sudo yum install ruby
ruby -v

cd /home/ec2-user

wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install

chmod +x ./install

sudo ./install auto

sudo service codedeploy-agent status
```

+만약 EC2 접속이 안된다면 보안그룹 인바운드 22포트를 열었는지 확인해주세요

### CodeDeploy
![[CI-CD CodeDeploy (1).png]]
![[CI-CD CodeDeploy (2).png]]

어플리케이션 생성 후 배포 그룹 생성을 클릭해줍니다.
![[CI-CD CodeDeploy (3).png]]

### deploy.yml
```
name: Build and Deploy Spring Boot to AWS EC2

on:
  push:
    branches: [main]

env:
  PROJECT_NAME: ci-cd-practice
  BUCKET_NAME: pack-ci-cd-bucket
  CODE_DEPLOY_APP_NAME: cd-cd-test
  DEPLOYMENT_GROUP_NAME: cd-cd-test-group

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

	  - name: Permission for deploy.sh  
		run: chmod +x ./scripts/deploy.sh

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

### appspec.yml
```
version: 0.0  
os: linux  
  
files:  
  - source: /  
    destination: /home/ec2-user/gdsc-ci-cd  
  
permissions:  
  - object: /home/ec2-user/gdsc-ci-cd/scripts/deploy.sh  
    mode: 755  
    owner: ec2-user  
    group: ec2-user  
  
hooks:  
  AfterInstall:  
    - location: scripts/deploy.sh 
      timeout: 60  
      runas: ec2-user
```

## Deploy.sh
```
#!/usr/bin/env bash

REPOSITORY=/home/ec2-user/[Repository Name]
LOG_DIR=$REPOSITORY/logs
LOG_FILE="$LOG_DIR/$(date +'%Y-%m-%d').log"

# 로그 디렉토리 생성 시 권한 문제 해결을 위해 sudo를 사용
if [ ! -d "$LOG_DIR" ]; then
  sudo mkdir -p "$LOG_DIR"
  sudo chown ec2-user:ec2-user "$LOG_DIR"
fi

cd $REPOSITORY

APP_NAME=[Repository Name] # 소문자로 작성해야됨
JAR_NAME=$(ls $REPOSITORY/build/libs/ | grep 'SNAPSHOT.jar' | tail -n 1)
JAR_PATH=$REPOSITORY/build/libs/$JAR_NAME

CURRENT_PID=$(pgrep -f $APP_NAME)

if [ -z $CURRENT_PID ]
then
  echo "> 종료할것 없음."
else
  echo "> kill -9 $CURRENT_PID"
  sudo kill -15 $CURRENT_PID
  sleep 5
fi

echo "> $JAR_PATH 배포"
nohup java -jar $JAR_PATH > $LOG_FILE 2>&1 &
```
## 오류
```
# 로그 디렉토리 생성 시 권한 문제 해결을 위해 sudo를 사용  
if [ ! -d "$LOG_DIR" ]; then  
  sudo mkdir -p "$LOG_DIR"  
  sudo chown ec2-user:ec2-user "$LOG_DIR"  
fi
```

디렉토리 생성 권한이 없어서 오류가 발생했었습니다. 

추가적으로 deploy.sh에 권한이 없어서 파일이 실행되지 않고 있었습니다.
따라서 deploy.yml에 다음을 추가해주었습니다.
```
- name: Permission for deploy.sh  
  run: chmod +x ./scripts/deploy.sh
```
