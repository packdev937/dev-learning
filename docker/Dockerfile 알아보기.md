1. Dockerfile는 최상위 루트 프로젝트 경로에 위치합니다.
2. 파일 이름은 Dokcerfile이라고 저장해야 합니다.
3. Dockerfile은 반드시 FROM 키워드로 시작해야 합니다. 하단의 명령어는 openjdk17 버전을 pull 받는 명령어 입니다.
```
FROM openjdk:17-jdk-slim-buster
```

4. ADD, COPY 키워드는 특정 위치에서 Docker 이미지로 파일을 복사하는 기능을 수행합니다
```
COPY ./build/libs/*.jar app.jar
```
위 명령어는  ./build/libs 디렉토리 내에 있는 모든 jar 파일을 `app.jar` 이라는 이름으로 복사합니다. 이 과정은 `docker build` 명령어를 실행할 때 수행됩니다. 

5. ENTRYPOINT 는 Docker 컨테이너가 시작될 때 실행되어야 할 기본 명령을 정의합니다. 지정된 명령은 컨테이너가 시작될 때마다 자동으로 실행되며, 컨테이너의 기본 실행 프로세스가 됩니다.
```
ENTRYPOINT ["java", "-jar", "/app.jar"]
```
예를 들어, 위 명령어는 컨테이너가 실행될 때 java -jar /app.jar 명령어가 자동으로 실행됩니다.

예시 Dockerfile은 다음과 같습니다.
```
FROM openjdk:17-jdk-slim-buster  
COPY build/libs/*.jar app.jar  
EXPOSE 8080  
ENTRYPOINT ["java", "-jar", "/app.jar"]
```
