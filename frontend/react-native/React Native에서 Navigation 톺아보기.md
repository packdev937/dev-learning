## 1. 개요 
React Native Expo에서 Navigation에 대해 조금 더 깊게 알아보겠습니다. 
## 2. 환경 설정 
React 에서는 React Router DOM 라이브러리를 활용하여 화면을 전환했다면, React Native 에서는 React Navigation 라이브러리를 사용합니다. 

패키지 설치는 다음 명령어를 입력합니다. 
```bash
$ npm install @react-navigation/native
```

추가적으로 Dependency를 설정해줍니다. 
```bash
$ npx expo install react-native-screens react-native-safe-area-context
```
## 3. Navigation의 종류
### 3-1. Stack Navigator 
화면 전환을 스택 구조로 관리합니다. 새로운 화면을 추가하면 스택에 쌓이고 뒤로 가기를 하면 스택에서 제거됩니다. 자연스러운 사용자 경험을 제공합니다.

```bash
$ npm install @react-navigation/stack
```

기본적인 예제 코드는 다음과 같습니다. 
```javascript
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

function MyStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Details" component={DetailsScreen} />
    </Stack.Navigator>
  );
}
```

### 3-2. Tab Navigator
탭 네비게이터는 화면 하단이나 상단에 탭을 배치하여 화면 간에 전환할 수 있도록 합니다. 일반적으로 앱의 주요 섹션을 탐색하는 데 사용됩니다

```bash
$ npm install @react-navigation/bottom-tabs
```

```javascript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

function MyTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
}
```

이쯤 되면 규칙이 보이시나요? `createXXXNavigator`를 만들고, 각 Screen은 Navigator로 감싸줍니다. Navigator와 Screen에 대한 자세한 내용은 다음 챕터에서 설명하겠습니다.

바텀 탭은 다음과 같이 생성됩니다. 
![[스크린샷 2024-08-06 오후 11.47.38.png]]
### 3-3. Drawer  Navigator
공식 문서로 대체합니다. 
https://reactnavigation.org/docs/drawer-based-navigation/

## 4. Navigation Container와 Navigator 
Navigation을 사용하고자 할 때는 App을 `<NavigationContainer`로 감싸주어야 합니다. 그리고 안에 각 종 Navigator `ex) Tab.Navigator, Drawer.Navigator, Stack.Navigator`를 사용하는데요, Navigator은 어떤 역할을 하는 걸까요? 

Navigator는 페이지 간의 전환을 관리하는 컴포넌트 입니다. 우리는 중첩된 Navigator를 통해 앱의 복잡한 화면 전환을 관리할 수 있습니다. 

예를 들어, 메인 화면의 탭에는 홈과 마이페이지가 있다고 가정해보겠습니다. 그리고 홈 페이지에서는 유저의 피드를 자세히 보거나, 자신의 피드를 올릴 수 있습니다. 

여기서 두 개의 네비게이터가 존재하는데, 첫 번째는 `<Tab.Navigator>`로 Home과 Mypage를 가지고 있습니다. 두 번째는 `<Stack.Navigator>` 로 마찬가지로 Home과 UserDetailFeed, UploadFeed 등이 있을 수 있습니다. (이름은 신경쓰지 않아도 됩니다) 

참고로 Tab의 Home과 Stack의 Home은 다릅니다. 
Tab의 Home은 `<Stack.Container>`를 가르키고 Stack의 Home은 `<Stack.Screen>`를 가르킵니다. 

대략적인 코드는 다음과 같을 수 있습니다.
```typescript
const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

function HomeStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="UserDetailFeed" component={UserDetailFeedScreen} />
      <Stack.Screen name="UploadFeed" component={UploadFeedScreen} />
    </Stack.Navigator>
  );
}

export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeStack} />
        <Tab.Screen name="Mypage" component={MypageScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

## 5. Type Checking with Type Script
Navigation을 사용할 때 타입 체크는 필요합니다.

먼저 타입 체크를 통해 각 화면에 전달되는 파라미터의 타입을 명확하게 정의할 수 있습니다. 예를 들어, 특정 화면이 특정 타입의 파라미터를 요구할 때, 잘못된 타입의 파라미터를 전달하는 오류를 사전에 방지할 수 있습니다.

예를 들어, `Profile` 화면에 `userId: string` 파라미터가 필요하다면, 타입 체크를 통해 `number`나 `boolean` 등 다른 타입의 값을 실수로 전달하는 것을 방지할 수 있습니다.

위 예제를 Type Checking 해보겠습니다.
```typescript
type HomeStackParamList = {
  Home: undefined; // Home 화면은 파라미터가 없습니다.
  UserDetailFeed: { userId: string }; // UserDetailFeed 화면은 userId 파라미터를 필요로 합니다.
  UploadFeed: undefined; // UploadFeed 화면은 파라미터가 없습니다.
};
```
참고로 `undefined`은  해당 route에 파라미터가 없다는 것을 의미합니다.

매핑을 정의했다면 우리는 Navigator 컴포넌트에 그것을 사용할 수 있도록 해주어야 합니다. 관련된 코드를 하나의 묶음으로 정의한다면 다음과 같습니다.
```typescript
type HomeStackParamList = {
  Home: undefined; // Home 화면은 파라미터가 없습니다.
  UserDetailFeed: { userId: string }; // UserDetailFeed 화면은 userId 파라미터를 필요로 합니다.
  UploadFeed: undefined; // UploadFeed 화면은 파라미터가 없습니다.
};

const Stack = createStackNavigator<HomeStackParamList>();

function HomeStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="UserDetailFeed" component={UserDetailFeedScreen} />
      <Stack.Screen name="UploadFeed" component={UploadFeedScreen} />
    </Stack.Navigator>
  );
}
```

### 5-1. 중첩 Navigator Type Checking 
`Tab.Navigator`에는 `Stack.Navigator`가 중첩되어 있습니다. TypeChecking 할 때도 이를 명시해줄 수 있습니다.
```typescript
type TabParamList = {
  Home: NavigatorScreenParams<HomeStackParamList>; // Home 탭은 HomeStack을 가리킵니다.
  Mypage: undefined; // Mypage 화면은 파라미터가 없습니다.
};

```

### 5-2. Screen에서 사용
```typescript
import { NativeStackScreenProps } from '@react-navigation/native-stack';

type Props = NativeStackScreenProps<HomeStackParamList, 'UserDetailFeed'>;

function UserDetailFeedScreen({ route, navigation }: Props) {
  // route.params.userId로 userId 접근 가능
  return (
    // JSX 코드
  );
}
```

## Reference
- https://reactnavigation.org/docs/typescript/