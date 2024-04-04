**도커 허브 로그인 상태 확인**
```bash
docker info | grep Username
```

로그인 되어 있으면 `Username` 이 출력되나, 로그인 되어있지 않다면 아무것도 출력되지 않습니다.

```bash
docker login

docker -u username 
```

로그아웃
```bash
docker logout
```

### 트러블 슈팅
![[스크린샷 2024-04-04 오후 1.25.58.png]]

config.json 파일을 엽니다. 
```bash
vi ~/.docker/config.json
```

다음 문구를 삭제합니다.
```python
"credStore": "desktop"
```
