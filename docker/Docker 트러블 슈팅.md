
```
ERROR: Couldn't connect to Docker daemon at http+docker://localhost - is it running?

If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.
```

docker 서비스를 재 시작해 줍니다.

```
WARNING: Found orphan containers (server) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Removing network ec2-user_default
ERROR: error while removing network: network ec2-user_default id 695bab4c5ac2cafc89d6dd7158156264fb799b8b2e5409ecda1edd2e22208a2f has active endpoints
```

docker-compose up으로 컨테이너를 올린 다음 docker-compose.yml 을 변경했습니다. 
따라서 기존 컨테이너가 고아가 되는 현상이 생겼는데, 이는 다음과 같이 해결할 수 있습니다.

``` bash
docker-compose down --remove-orphans
```