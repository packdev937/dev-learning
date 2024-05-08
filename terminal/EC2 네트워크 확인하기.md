NGINX 설정 도중 문제가 생겨 포트가 제대로 열려있나 확인할 필요가 있었습니다.
```null
sudo netstat -plant | grep '80'
```

```null
sudo netstat -plant | grep '8080'
```

### Reference
- https://velog.io/@nche/Ngnix-connect-failed-111-Connection-refused-while-connecting-to-upstream