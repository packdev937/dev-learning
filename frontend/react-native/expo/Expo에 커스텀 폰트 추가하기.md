## 1. 개요 
디자인 요구 사항 중 하나인 `Pretendard` 폰트를 개발 중인 Expo 앱에 적용해보려고 합니다. 

## 2. Setup 
먼저 다음 링크를 통해 폰트를 다운로드 받아 줍니다. 
[Pretendard 다운로드](https://cactus.tistory.com/306)

압축 파일을 다운로드 받게 되는데, 압축 해제 후 `Pretendard-Medium.otf` 을 `assets/fonts/` 로 복사해줍니다. 
![[스크린샷 2024-10-14 오후 2.53.48.png|500]]

다음으로 라이브러리 설치를 진행해야 합니다.
```bash
$ npx expo install expo-font
```

`expo-font` 라이브러리를 사용하여 애플리케이션에서 커스텀 폰트를 로딩하고 이를 적용할 수 있습니다. 

라이브러리 설치 후 `app.json`에 이를 명시할 필요가 있습니다. 
```
{
	"expo" : {
		"plugins" : [
			["expo-font",  
				{  
				    "fonts" : ["./assets/fonts/Pretendard-Medium.otf"]  
				}  
			]
		]
	}
}
```

적용된 폰트는 다음과 같이 사용 가능합니다. 
```js
export default function App() {
	const [isFontLoaded] = useFonts({  
	  Pretendard: require('@/../assets/fonts/Pretendard-Medium.otf'),  
	})  
	  
	if(!isFontLoaded) {  
	  return (  
	    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>  
	      <ActivityIndicator size="large" />  
	    </View>  
	);  
}

...

const styles = StyleSheet.create({
  buttonText: {  
    color: '#FFFFFF',  
    fontSize: 16,  
    fontFamily: 'Pretendard'  
  },  
});
```
## Reference
https://docs.expo.dev/versions/latest/sdk/font/