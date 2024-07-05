React Native 에서 ios 앱을 빌드 하기 위해서는 다음과 같은 절차가 필요합니다. 
1. Node.js 설치 
2. xCode 설치 
3. Cocoapods 설치 

Cocoapods 설치 중 문제가 발생한다면 다음 명령어를 입력합니다. 
```bash
sudo gem install cocoapods -n /usr/local/bin
```

```
sudo gem install bundler -v 2.2.3 -n /usr/local/bin
```

다음과 같은 xCode 에러가 발생한다면 
```
Error: Error: Command failed with exit code 1: xcodebuild -list -json xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance
```

Active Developer 경로를 다시 지정해줍니다.
```
sudo xcode-select -s /Application/Xcode.app/Contents/Developer
```

초반에 떠야 하는 다음과 같은 화면이 뜨지 않았습니다.
![[스크린샷 2024-07-04 오후 8.22.07.png|400]]

따라서 다음 명령어를 실행해 캐시를 지워주었습니다.
```bash
watchman watch-del-all
rm -rf node_modules
npm install
npm start -- --reset-cache

```

iPhone 시뮬레이터 실행 중 다음과 같은 에러가 발생하였습니다.
![[스크린샷 2024-07-04 오후 9.58.01.png|300]]
npx react-native start 로 하니 정상적으로 실행됐습니다.

- `npx react-native start`: Metro Bundler를 시작합니다.
- `npx react-native run-ios --simulator="iPhone 15"`: iOS 시뮬레이터에서 앱을 빌드하고 실행합니다.

시도한 첫 번째 방법은 `ios/build` 디렉토리를 삭제하는 방법입니다. 
```
rm -rf ios/build
```

![[스크린샷 2024-07-04 오후 10.04.40.png]]
finder에서 launchPackager.command 파일을 찾은 뒤 다음으로 열기 -> iterms 후 항상 선택한 응용프로그램으로 열기를 체크 합니다. 

`Unable to boot device in current state: Booted` 가 뜬다면 시뮬레이터 재부팅을 해줍니다.
```
xcrun simctl shutdown all
```

필요에 따라 시뮬레이터를 재설정합니다.
```
xcrun simctl erase all
```