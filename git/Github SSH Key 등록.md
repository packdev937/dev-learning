우선 등록된 SSH 키가 없는지 확인해야 합니다. 
```shell
cat ~/.ssh/id_rsa.pub
```

없다면 다음 명령어를 통해 SSH 키를 생성합니다. 
```shell
ssh-keygen
```

명령어를 성공적으로 실행했다면 두 개의 파일이 생성됩니다. id_rsa, id_rsa_pub 이며 각각 개인 키, 공개 키를 의미합니다.

![[id_rsa.png]]

이제 깃허브로 들어가서 SSH 키를 등록해주어야 합니다.
![[스크린샷 2024-03-14 오전 10.07.58.png]]

참고로 다음 명령어를 사용하면 파일 내에 있는 내용을 자동으로 클립보드로 복사합니다.
```shell
pbcopy < ~/.ssh/id_rsa.pub
```

이제 해당 SSH 키를 사용해서 Git Clone을 진행해봅시다.
```shell
git@github.com:DigitalPharm/dp-dtx-001-backend.git
```
