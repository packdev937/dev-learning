보통 민감한 정보들은 .gitignore에 경로 혹은 파일 형식을 작성하여 커밋 되지 않게 설정합니다.

간혹 작업 이후에 .gitignore에 추가하는 경우가 있는데 이 때, git이 이미 추적 중인 파일은 `.gitignore`에 추가한다고 자동으로 무시되지 않습니다. 이런 파일들을 Git 추적목록에서 제거하기 위해 캐시를 삭제해야 합니다. 다음 명령어를 사용하여 `.gitignore`에 명시된 무시 패턴에 일치하는 파일을 캐시에서 삭제할 수 있습니다:

```
git rm -r --cached .
git add .
git commit -m "캐시 삭제 후 .gitignore 적용"
```
