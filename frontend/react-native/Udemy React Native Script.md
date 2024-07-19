## Section 1

### ★★★ [Create React Native Project ](https://reactnative.dev/docs/environment-setup)
- To install *EXPO PACKAGE*, downloading node.js first.
#### Quick Start 
- `npm install --g expo-cli`
	
	이 때 나오는 에러들은 무시해도 무방합니다. 
	![[스크린샷 2024-07-16 오후 2.56.59.png]]
	
	Expo 명령어를 통해 Expo가 잘 설치되었는지 확인하세요! 에러가 안나온다면 정상적으로 설치된 것입니다. 
	![[스크린샷 2024-07-16 오후 2.57.57.png]]
	[Expo에 대해 더 알고 싶다면](https://expo.dev/)
- `npx create-expo-app@latest [My Project Name]` 
	설치가 완료됐다면 Expo를 실행하기 위한 몇 가지 명령어가 나타납니다
	![[스크린샷 2024-07-16 오후 3.01.56.png]]

### Expo Project Structure 
- `package.json`을 보면 expo 패키지가 의존성 등록되어 있는 것을 확인할 수 있습니다. expo 패키지는 React Native 코드를 쉽게 작성할 수 있는 유틸리티 기능을 많이 제공합니다. 
![[스크린샷 2024-07-16 오후 3.05.34.png]]
- `bable.config.js` 파일은 코드가 내부적으로 어떻게 변환되는지 설정합니다. [바벨에 대해 더 자세히 알아보기](https://xtring-dev.tistory.com/entry/Babel-%EB%AA%A8%EB%A5%B4%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8D%98-Babel-%EC%9D%B4%EC%A0%A0-%EB%91%90%EB%A0%A4%EC%9B%8C-%EB%A7%88%EC%84%B8%EC%9A%94-Babel-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
- `app.json` 파일에서는 앱의 설정과 실행 방식을 구성합니다. 앱의 미리보기나 실제 앱스토어에 빌드할 때 Expo에 이어서 사용할 파일입니다. 이름이나 배경색 등을 설정할 수 있습니다. 

### Expo 실행하기 
- `npm start` 명령어를 통해 Expo 개발 서버를 실행합니다.
- 더 자세한 내용은 [다음](https://docs.expo.dev/get-started/start-developing/)을 확인합니다.
	제대로 실행이 됐다면 QR 코드가 뜹니다. 
	코드 작업 중에는 npm start를 유지해야 합니다. 
	![[스크린샷 2024-07-16 오후 3.18.25.png]]
- 참고로 `App.js`가 없어 별도로 생성해주었습니다. 
```javascript
import { StatusBar } from 'expo-status-bar';  
import { StyleSheet, Text, View } from 'react-native';  
  
export default function App() {  
  return (  
    <View style={styles.container}>  
      <Text>Open up App.js to start working on your app!</Text>  
      <StatusBar style="auto" />  
    </View>  );  
}  
  
const styles = StyleSheet.create({  
  container: {  
    flex: 1,  
    backgroundColor: '#fff',  
    alignItems: 'center',  
    justifyContent: 'center',  
  },  
});
```

- `command + d` 를 눌러 developer tools을 열 수 있습니다. 
	`JS Debugger`를 연 모습 
	![[스크린샷 2024-07-16 오후 3.38.16.png]]
	Element Inspector를 연 모습 
	![[스크린샷 2024-07-16 오후 3.38.57.png]]
- `npm run reset-project`로 boilderplate code를 제거할 수 있습니다. 
	화면이 그대로라면 캐시가 남아있을 가능성이 존재합니다. 
	`expo start -c
## Section 2
### Core Component 
- `<View>` 는 일반적으로 콘텐츠를 담는 상자나 컨테이너 구축에 사용됩니다.
- `<Text>`, `<TextInput>`, `<Image>` 등을 포함할 수 있습니다. 하지만 단지 텍스트 만을 포함할 수는 없습니다.
- `<View>`는 컴포넌트를 담는 데 사용하는 유일한 컨테이너 컴포넌트 입니다.
- `StyleSheet` 객체는 React Native에서 가져온 내장 객체입니다. 

### Flexbox 
- `flex : 1` 속성은 사용할 수 있는 모든 공간을 차지하도록 지시하는 것입니다.
- `flexDirection`은 요소를 column (default) 혹은 row로 배치할 지 결정합니다.
- `justfiyContent`, `alignItem`은 요소를 각 축에 어떻게 배치할 지 제어할 수 있습니다. 
	- `justifyContent`는 주축(main axis)을 따라 자식 요소들을 어떻게 배치할지 제어합니다. 기본값은 `flex-start`입니다. 옵션들은 다음과 같습니다:
		- `flex-start`: 자식 요소들을 주축의 시작점으로 정렬합니다.
		- `flex-end`: 자식 요소들을 주축의 끝점으로 정렬합니다.
		- `center`: 자식 요소들을 주축의 중앙으로 정렬합니다.
		- `space-between`: 자식 요소들 사이에 동일한 간격을 둡니다.
		- `space-around`: 자식 요소들 주위에 동일한 간격을 둡니다. (요소 양쪽에 반 간격씩 적용)
		- `space-evenly`: 자식 요소들 사이와 주위에 동일한 간격을 둡니다.
	- `alignItems`는 교차축(cross axis)을 따라 자식 요소들을 어떻게 배치할지 제어합니다. 기본값은 `stretch`입니다. 옵션들은 다음과 같습니다:
		- `flex-start`: 자식 요소들을 교차축의 시작점으로 정렬합니다.
		- `flex-end`: 자식 요소들을 교차축의 끝점으로 정렬합니다.
		- `center`: 자식 요소들을 교차축의 중앙으로 정렬합니다.
		- `stretch`: 자식 요소들의 크기를 교차축에 맞게 늘립니다.
		- `baseline`: 자식 요소들의 텍스트 베이스라인을 기준으로 정렬합니다.

- 플렉스 박스에 있는 모드 자식 View는 부모 요소의 높이를 취하지만 넓이는 영향을 받지 않습니다. 
	- 예를 들어, 부모 View의 style에 height: 300, weight: '80%'로 적용한다면 height만 적용되어 길다란 선이 표현됩니다.
- 그건 기본 값의 설정 때문인데요, 교차 축에 대해 자세히 알아야 합니다. 
- 주축은 flexDirection의 값에 의해 정해지고 교차축은 주축의 반대입니다. 

- 배열을 병합하는 과정에서 [...toDos, todo] 는 상태를 업데이트하는 좋은 방법이 아닙니다.
- 좋은 방법은 다음과 같습니다.
```javascript
function addGoalHandler() {
	setCourseGoals((currentCourseGoals) => [
		...currentCourseGoals, 
		enteredGoalText
	])
}
```
- 즉, 이전 상태를 참조해서 업데이트하는 것입니다. (currentCourseGoals)에 자동으로 이전 상태가 매개변수로 넘어옵니다. 

- 각 플랫폼 (ios, android) 마다 스타일링의 차이가 존재할 수 있습니다. 예를 들어, ios의 경우 `<Text>`에 둥근 모서리가 적용 안됩니다. 따라서 `<View>`로 감싼 뒤 스타일을 `<View>`에 적용하면 둥근 모서리를 적용할 수 있습니다.

- `<ScrollView>`의 스크롤 가능한 영역은 부모 요소가 결정합니다. 즉, `<ScrollView>`를 `<View>`로 감싼 다음에 `<View>`에서 차지할 공간을 명시하고 `<ScrollView>`는 해당 공간에 항목들을 스크롤 할 수 있게 해주는 것입니다.
```r
<View style = {styles.container}>
	<ScrollView>
		...
```
- 방대한 구성 요소를 기억하고 어떤 방법을 사용하는 것이 좋을까에 대한 고민을 하는 것은 매우 중요합니다.
- `<ScrollView>`는 모든 요소를 렌더링 하기 때문에 대용량 데이터를 처리하기에 부적합합니다. 이 때, `<FlatList>`가 대체할 수 있습니다.
- `<FlatList>`는 data 속성이 존재합니다. 해당 속성에는 출력할 데이터를 전달합니다. 
- `<FlatList>`는 Self Closing으로 닫아 준 뒤, renderItem 속성에 화살표 함수로 컴포넌트를 정의합니다. 
```r
<FlatList
	data = {courseGoals}
	renderItem = {(item) => {
		return (
			<View> <Text> </Text> </View>
		);
	}}
	alwaysBounceVertical = {false}
/>
```

- 컴포넌트를 작은 컴포넌트로 쪼개는 것이 중요합니다. 

- 상황 예시를 들어보겠습니다. 투두 앱에는 `App.js`가 있고 `GoalInput.js`와 `GoalText.js`가 있습니다. `App.js`는 모든 것을 총괄하고 있기 때문에 ToDos 리스트를 가지고 있고 이를 `<GoalText/`로 넘겨주어야 합니다. `GoalInput.js`에서 입력을 다 처리하기 때문에 useState는 GoalInput에 존재해야합니다. 하지만 `App.js`에는 다음과 같은 함수가 존재합니다.
```javascript
function addGoalHandler() {
	setCourseGoals((currentCourseGoals) => [
		...currentCourseGoals,
		{ text : enteredGoalText, id : Math.random().toString() },
	]);
}
```

- 즉, `GoalInput.js`에서 입력한 Text를 다시 `App.js`로 넘겨줘야 합니다. 방법이 있을까요? 
```javascript
function addGoalHandler() {
	props.onAddGoal(enteredGoalText);
}
```
- props로 함수를 넘길 수도 있습니다. 해당 onPress 이벤트 트리거를 바로 반환하는게 아닌 조금의 Custom 과정을 거쳐서 매개변수를 넘긴채 넘겨주는 것입니다. 
```javascript
function addGoalHandler(enteredGoalText) {
	...
}
```

## Section 3
- 개발 시작 시 어떤 화면에 어떤 컴포넌트가 들어갈지 고민해봅시다. 
- `<Button>`도 결국 `<View>`와 `<Text>`의 조합입니다.
- Props로 넘어오는 값도 객체 비구조화를 할 수 있습니다.
```javascript
function PrimaryButton(props) {
	return (
		<View>
			<Text>props.children</Text>
		</View>
	)
}
```

```javascript
function PrimaryButton({ children }) {
	return (
		<View>
			<Text>children</Text>
		</View>
	)
}
```

- React Native에서 가장 먼저 로딩하는 파일을 `App.js` (혹은 `Index.tsx`) 이기 때문에, 해당 파일에서 내보낸 컴포넌트가 메인 컴포넌트로 랜더링 됩니다.
- `flex : 1`은 해당 객체가 최대한 많은 공간을 차지하게끔 합니다. 만약 해당 속성이 없다면 요소를 포함할 정도의 공간으로 줄어들게 됩니다.
- 그림자를 넣는 방법은 웹과 다릅니다. `Android`의 경우 styles에 elevation 프로퍼티를 추가합니다. ios의 경우 shadow로 시작하는 속성을 사용해야 합니다. 
```
const styles = StyleSheet.create({
	inputContainer : {
		// for Android
		elevation : 4,

		// for ios
		shadowColor: 'black',
		shadowOffset: {width: 0, height: 2},
		shadowRadius: 6, 
		shadowOpacity: 0.25
	}
})
```

- 여러 값을 사용해보고 맞는 스타일링을 찾는 것이 중요합니다. 예를 들어, 숫자는 1~99 사이의 값만 입력된다고 했을 때 굳이 넓은 칸의 스타일을 줄 필요는 없습니다. 
- `<TextInput>`의 `maxLength` 속성을 통해 입력 값 길이의 제한을 둘 수 있습니다. 
- `keyboardType`을 사용해서 어떤 키보드가 나오는지 설정할 수 있습니다. 기기에 마다 다르니 어떤 플랫폼이 어떤 키보드를 지원하는지 확인할 필요가 있습니다.
- `textAlign` 속성을 통해 텍스트를 컨테이너 중앙에 위치시킬 수 있습니다. 
- `overflow : 'hidden'` 은 컨테이너 내부에 일어나는 효과나 스타일링이 컨테이너를 넘어가는 부분을 잘라서 안보이도록 만듭니다.  
- `Pressable` 컴포넌트의 스타일 속성은 화살표 함수 같은 함수를 가지며, Pressable 이 눌릴 때 마다 자동으로 호출됩니다. 누르는 이벤트가 발생할 때 마다 이벤트에 관한 데이터 `pressed`가 호출되며 해당 `pressed`를 통해 여러 스타일을 조정할 수 있습니다. 
```javascript
<Pressable
	style = {(pressed} => pressed ? ~ : ~)}
```
- 예를 들어, 눌렀을 때 투명도가 있게 끔 설정하면 애니메이션과 같은 효과를 줄 수 있습니다. 
- [A, B] 를 통해 두 A, B를 통합할 수 있습니다. style에 적용할 수 있습니다.

- `alignItems: 'stretch'`로 설정하면 따로 `width` 혹은 (`height`)를 지정하지 않는 한 최대한으로 크기가 늘어납니다.
- `<View>`를 여러 레벨로 중첩해서 레이아웃을 완성합니다. 예를 들어, `<TextInput>`과 동일한 레벨에 있던 버튼들을 양 옆에 두고 싶어 별도의 `<View>`를 만들고 `flexDirection: 'row'`로 바꿨고, 각 버튼의 크기를 동일하게 만들고 싶어 각 버튼을 `<View>`로 감싼 뒤 `flex : 1` 이라는 동일한 스타일을 추가해주었습니다. 

- 루트 컴포넌트 (`App.js`) 에서 설정한 backgroundColor 색은 하위의 모든 `<View>`의 backgroundColor가 됩니다. 
	- `return <GameScreen/>`을 `return <View> <GameScreen/> </View>`로 감싼 이유
	- `<View`를 루트로 인식하게 만들고 해당 View에 배경 색 스타일을 적용하기 위해 
	- 참고로 `<View>`는 표현하는 공간만 차지 합니다. 만약에 전체 화면을 꽉채우고 싶게 한다면 루트 스타일을 `flex : 1`로 설정해야 합니다.

- `<LinearGradient>`를 사용하면 선형 그라데이션 배경을 지정할 수 있습니다. 별도의 expo 라이브러리 설치가 필요합니다. 
- `<ImageBackground>`를 통해서 그라이데이션과 이미지를 잘 조합할 수도 있습니다. (with 투명도 설정) 

- 컴포넌트로 부터 반환 값을 받는 방법은 다음과 같습니다. 
```javascript
function pickNumberHandler(pickedNumber) {
	setUserNumber(pickedNumber)
}

let screen = <StartGameScreen onPickNumber = {pickedNumberHandler} />

if(pickedNumber) {
	screen = <GameScreen />
}
... 

function StartGameScreen({ onPickNumber }){
	...
	onPickNumber(10);
}
```
- 함수의 참조를 이용해서 값을 설정하는 느낌이라고 보면 됩니다.

- `<SafeAreaView>`를 통해 기기 화면 제한을 고려할 수 있습니다.

- 앱 내에 여러 위치에서 사용하는 요소가 있을 때는 재사용성을 고려한 분리를 생각해봐야 합니다. 예를 들어, 앱의 상단의 Title의 경우 별도의 `Title.js`로 분리하는 것을 고려할 만 합니다.  

- 반복되는 상수 (색상) 등은 별도의 상수 클래스를 만드는 것도 하나의 방법입니다. 

- `bind()`을 사용해서 함수 실행에 사용될 매개변수의 값을 사전 구성할 수 있습니다. 
```javascript
function nextGuessHandler(direction) {
	if (direction === 'lower')
		...
}

<PrimaryButton onPress = {nextGuessHandler.bind(this, 'lower')} > ...
```
- 첫 번째 매개변수로는 this를 넘겨주어야 합니다.

- useEffect()가 리랜더링해서 문제를 야기할 수 있습니다. 예를 들어 숫자 맞추기 게임에서 하한값, 상한값, 그리고 userNumber을 매개변수로 넘겨줍니다. 초기에는 1, 100, 그리고 유저가 입력한 번호로 설정되어 큰 문제가 없지만, 후에 숫자를 다 맞추고 나면 다시 한번 리랜더링 되고 이 과정에서 문제가 발생합니다.
```javascript
function GameScreen({ userNumber, onGameOver }) {
	const initialGuess = generateRandomBetween(
		min,
		max,
		userNumber	
	); // 맞춘 시점에는 min === max 일 것이고, 이 때 재 랜더링하면 오류가 난다 

	useEffect(() => {
		if(currentGuess === userNumber) {
			onGameOver();
		}
	}, [currentGuess, userNumber, onGameOver]);
}
```

- props는 두 가지 방식으로 전달될 수 있습니다.
	 - `<Card title="Hello" />`처럼 사용하면 `title`이라는 속성으로 전달됩니다.
	- `<Card>Hello</Card>`처럼 사용하면 `Hello`가 `children` prop으로 전달됩니다.
- 태그 안에 있는 요소가 props로 넘어가게 됩니다.
- 추가적으로 Custom 컴포넌트에 style을 props으로 넘겨서 다음과 같이 추가할 수도 있습니다. 
```r
import React from 'react';
import { Text } from 'react-native';

function Card({ children , style}) {
    return <Text style = {[styles.container, style]}>{children}</Text>;
}

const styles = StyleSheet.create{[
	container : {
		...
	}
]}

function App() {
    return (
        <div>
            <Card>Hello</Card>
        </div>
    );
}

export default App;

```

★
- `@expo/vector-icons`를 통해 쉽게 아이콘을 불러올 수 있습니다.
- `icons.expo.fyi.` 공식 문서를 통해 내장된 아이콘 리스트들의 이미지를 볼 수 있습니다.

★
- 앱의 루트 컴포넌트에 커스텀 폰트를 설정하는게 좋습니다. 앱이 시작될 때 로딩이 시작되는게 (빠르게) 좋기 때문입니다. 
- `expo install expo-font` 패키지를 설치한 뒤, `useFonts` 훅을 불러옵니다. 이는 로딩하기 위해 App 컴포넌트에 무조건 import 해야됩니다. 
- `@expo-google-fonts/inter`를 통해 구글 폰트를 쉽게 받을 수도 있습니다. 
- `/assets/fonts`에 로컬 폰트를 다운 받아도 무방합니다. 
- `useFonts` 객체에 로딩한 글꼴을 구별하는데 쓰일 프로퍼티 이름을 설정하고, 값을 설정하여 로딩할 파일을 지정해야 합니다.
- 일반적으로 글꼴이 로딩되는 동안 스플래시 화면을 띄웁니다. 
- 글꼴을 fontFamily 속성을 통해 사용 가능합니다.
```r
const [fontsLoaded] = useFonts({
	'open-sans',: require('./assets/fonts/OpenSans-Regular.ttf'),
	'open-sans-bold',: require('./assets/fonts/OpenSans-Bold.ttf'),
})
```
- - 가 있으면 ' 작은 따옴표를 쓰세요.
- `expo install expo-app-loading` 의 유틸 함수를 통해 스플래시 화면을 연장하고 특정 조건을 충족하기 전까지 스플래시 화면이 나오게 합니다.
- 폰트가 로딩이 되었다면 컴포넌트가 재실행되어 스플래시 화면이 나타나지 않게 됩니다.
```r
if(!fontsLoaded) {
	return <AppLoading />;
}
```

- Image도 똑같이 `overflow` 속성을 사용해서 스타일링 합니다. 
```
<View style = {styles.imageConatiner}>
	<Image style = {styles.image} 
	source = {require('../[path])} />
</View>

const styles = StyleSheet.create({
	imageContainer : {
		width : 300, 
		height : 300, 
		borderRadius : 200, 
		borderWidth : 3, 
		borderColor : ~,
		overflow : 'hidden'
	},
	image : {
		width : 100%, 
		height: 100% 
	}
})
```

★
- 참고로 크기는 사용 가능한 기기에 맞춰 동적으로 조정하는 작업이 필요합니다. 

- `<Text>`는 또 다른 `<Text>`로 감싸는 것이 가능합니다. 
- 중첩된 `<Text>`에 다른 스타일링을 추가할 수도 있습니다. 예를 들어, 특정 텍스트를 강조할 수 있습니다.
- 부모 텍스트 컴포넌트에 설정한 스타일은 자식 텍스트 컴포넌트에 영향을 미칩니다.

- 숫자 맞추기 게임의 경우 게임이 시작될 때 마다 상한값, 하한값을 초기해야 합니다. 다만, 게임이 진행될 때는 초기화 되면 안됩니다. 이는 useEffect()를 통해 설정할 수 있습니다. 
- useEffect()는 컴포넌트가 재 랜더링 될 때 실행되기 때문입니다. 그리고 의존성 배열을 빈 배열로 넘겨준다면 처음 컴포넌트가 실행될 때만 실행됩니다. 
```r
useEffect(() => {
	min = 1;
	max = 100;
}, []);
```

- `FlatList` 부모 컨테이너를 통해 높이를 조절하면 기기의 경계를 벗어나지 않는 선에서 스크롤 할 수 있습니다.

## Section 5 
- 다양한 플랫폼과 여러 기기 크기에 따라 조정되는 사용자 인터페이스 경험은 매우 중요합니다.
- 적응형 컴포넌트 

- `maxWidth : '80%'` 속성은 기기에 맞춰 최대 80% 만 공간을 활용합니다. 이는 부모 컴포넌트가 다루는 크기의 80%를 의미합니다.
- width와 maxWidth를 결합해서 사용하는 것은 매우 유용합니다.

★
- 값을 하드 코딩하는 것은 좋지 않습니다.
- 퍼센트로 하게 되면 너비와 높이는 달라질 수 밖에 없는데, 기기의 크기에 따라 다른 값으로 하드 코딩 된다면 미연에 여러 문제들을 방지할 수 있습니다.
- Dimensions API를 사용하려면 `react-native`에서 import를 해야 합니다. 
- Dimensions API에서 기기의 너비와 높이를 추출할 수 있습니다.
```javascript
const deviceWidth = Dimensions.get('screen').width;
const deviceHeight = Dimensions.get('window').height;
```
- 대게 수학식이나 삼항연산자를 사용합니다.
```
padding : deviceWidth < 450 ? 24 : 12;
```
- 이는 `fontSize`, `margin` 등에 이용될 수 있습니다.

- `app.json`에 `orientation : "portrait"`으로 되어 있다면회전이 되지 않습니다. 
- 앱에 따라 고정하지 않는 경우도 많습니다. 
- 값을 "default"로 바꾸면 됩니다.
- 사용 중 변환하고 싶다면 Dimensions API는 사용하지 않는게 좋습니다. 도중에 값이 재 설정되지 않기 때문입니다.
- 이 경우 반응형 코드를 사용하는 것이 좋습니다. 
- 기기의 방향이 바뀔 경우 컴포넌트 함수가 재 실행되고 너비와 길이가 업데이트 됩니다.
- 즉, 컴포넌트 함수 내에서 동적 높이 혹은 너비를 사용할 수 있다는 것입니다.
```r
const { width, height } = useWindowDimensions();

const marginTopDistance = height < 300 ? 30 : 100;
```

- ios에서는 키보드가 화면을 초과했을 때  키보드를 닫는 화면도, 배경을 클릭해도 나가지지 않고, 입력도 되지 않습니다. 
- `KeyboardAvoidingView`를 사용해야 합니다. 
- 키보드가 열릴 때 마다 해당 요소가 제일 위로 올라가 키보드가 열려도 엑세스 할 수 있도록 해줍니다. 
- rootContainer가 포함된 뷰를 `<KeyboardAvoidingView>`로 감싸줍니다. 
- 하지만 적절한 스타일을 설정해주지 않으면 화면은 깨집니다. 
- 또한 `behavior = "position"`을 통해 속성을 지정할 수 있지만 제대로 사용하려면 `<ScrollView>`로 감싸는 것이 좋습니다. 
```r
<ScrollView style = {styles.screen}>
	<KeyboardAvoidingView style = {styles.screen} behavior = "position">
		<View ...>
```

- 각 모드에 대응하는 방법 중 하나는 아예 별도의 UI를 만들어서 제공하는 것입니다. 
- `useWindowDimensions()`를 통해 얻은 `width`, `height` 값을 통해서 가로, 세로 모드를 판단할 수 있습니다.

## Section 6
- 특정 화면으로 이동하거나 이전 화면으로 되돌아가기 위해서는 `Navigation`이 필요합니다. 
- 웹의 `Navigation` URL을 입력해서 특정 페이지에 도달하고 링크를 사용해서 하위 페이지로 이동하는 개념입니다. 
- 모바일에서는 버튼을 눌러서 한 화면에서 다른 화면으로 이동하거나 되돌아가는 것을 의미합니다. 

- 고정된 리스트 목록에서는 `<FlatList>`가 필요하지 않을 수 있습니다. 
```r
function renderCategoryItem(item) {
	return;
}

function CategoriesScreen() {

	<FlatList data = {CATEGORIES} 
	keyExtractor={(item) => item.id} 
	renderItem={renderCategoryItem}
	numColumns = {2} />

}
```
- renderItem 프로퍼티를 추가해서 각 항목에 실행될 함수를 정의하고 항목을 랜더링 하는 방법을 알려줘야 합니다.
★
- `numColums` 속성은 후에 사진을 표현할 때 좋을 듯 합니다.
- `Platform.OS === 'andorid' ? 'hidden' : 'visible'`
- ios `Pressable`에 스타일을 추가하는 방법은 다음과 같습니다.
```r
<Pressable
	style = {({ pressed }) => [styles.button, pressed ? styles.buttonPrssed : null ]}
```

- `npm install @react-navigation/native`
- `expo install react-native-screens react-native-safe-area-context`
- `<NavigationContainer>`로 감싸주어야 합니다.
- 다양한 Navigator가 존재합니다. 
	- Stack, Native Stack, Drawer, Bottom Tabs, Material Bottom Tabs
- `npm install @react-navigation/native-stack`
```r
import { createNativeStackNavigator } from @react-navigation/native-stack`

// 컴포넌트 밖에 선언
const Stack = createNativeStackNavigator();

export default function App() {
	<NavigationContainer>
		<Stack.Navigator>
			<Stack.Screen name = {} component = {}/>
			...
}
```