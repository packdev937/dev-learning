WebStorm에서 `Command + Option + L`을 눌러 코드를 포맷할 때 Prettier 설정이 적용되지 않는 문제를 해결하려면 다음 단계를 따라 설정을 확인하고 조정해보세요.

### 1. Prettier 플러그인 설치
WebStorm에서 Prettier 플러그인이 설치되어 있는지 확인합니다.
1. `Preferences` (`Command + ,`)를 엽니다.
2. `Plugins`로 이동합니다.
3. Prettier 플러그인이 설치되어 있는지 확인하고, 설치되어 있지 않다면 설치합니다.

### 2. Prettier 설정 확인
Prettier 설정이 올바르게 구성되어 있는지 확인합니다.
1. `Preferences` (`Command + ,`)를 엽니다.
2. `Languages & Frameworks` > `JavaScript` > `Prettier`로 이동합니다.
3. `Prettier package` 경로가 올바르게 설정되어 있는지 확인합니다.
4. `On code reformat` 및 `On save` 옵션이 체크되어 있는지 확인합니다.

### 3. File Watchers 설정
WebStorm의 File Watchers를 사용하여 파일이 저장될 때 Prettier를 자동으로 실행하도록 설정할 수 있습니다.
1. `Preferences` (`Command + ,`)를 엽니다.
2. `Tools` > `File Watchers`로 이동합니다.
3. `+` 버튼을 클릭하고 `Prettier`를 선택합니다.
4. 파일 패턴을 `*.js`와 `*.jsx`로 설정합니다.

### 4. ESLint 설정 확인
ESLint 설정이 Prettier와 호환되도록 설정되어 있는지 확인합니다.
1. `.eslintrc.js` 파일을 아래와 같이 구성합니다.
   ```javascript
   module.exports = {
     root: true,
     extends: ['@react-native', 'plugin:prettier/recommended'],
     plugins: ['prettier'],
     rules: {
       'prettier/prettier': 'error',
     },
   };
   ```

### 5. WebStorm에서 External Tools 설정
Prettier를 External Tools로 설정하여 단축키를 통해 실행할 수 있도록 합니다.
1. `Preferences` (`Command + ,`)를 엽니다.
2. `Tools` > `External Tools`로 이동합니다.
3. `+` 버튼을 클릭하여 새로운 도구를 추가합니다.
   - Name: Prettier
   - Program: `node_modules/.bin/prettier`
   - Parameters: `--write $FilePathRelativeToProjectRoot$`
   - Working directory: `$ProjectFileDir$`

### 6. External Tools 단축키 설정
External Tools에 단축키를 설정하여 Prettier를 실행할 수 있도록 합니다.
1. `Preferences` (`Command + ,`)를 엽니다.
2. `Keymap`으로 이동합니다.
3. `External Tools` > `Prettier`를 찾습니다.
4. 해당 항목에 오른쪽 클릭하여 `Add Keyboard Shortcut`을 선택합니다.
5. 원하는 단축키를 설정합니다 (예: `Command + Option + P`).

위의 단계를 따라 설정하면 WebStorm에서 Prettier가 제대로 작동하고 `Command + Option + L`을 눌렀을 때 코드를 Prettier 설정에 따라 포맷할 수 있을 것입니다. 

단축키 설정이 제대로 되었는지 확인하고, 필요한 경우 Prettier가 적용된 파일을 수동으로 포맷하여 작동을 확인합니다.