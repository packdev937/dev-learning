## 1. 개요
React Native Navigation에 대해 찾아보던 도중 (tabs) 라는 키워드가 나와서 찾아보았습니다. 많은 강의들은 `App.js`를 루트 파일로 사용하는데, app/_layout.tsx가 어떤 역할을 하는지 알아보도록 하겠습니다.

![[스크린샷 2024-08-07 오전 12.12.14.png]]

*시작에 앞서 typescript가 expo의 default 언어가 되었다는 것을 짚고 넘어가겠습니다.* 
## 2. File-Based Routing
> 파일 기반 라우팅은 디렉토리 구조와 파일명을 기반으로 라우팅을 자동으로 설정하는 방식입니다.

기본 명령어로 expo 패키지를 만들면 다음과 같은 defualt 구조가 생성되게 됩니다. 
![[Pasted image 20250105131156.png|200]]

##### `app/_layout.tsx`의 역할 
`_layout.tsx` 파일은 해당 디렉토리와 하위 디렉토리의 **공통 레이아웃을 정의**합니다.
- `app` 디렉토리의 **공통 UI 구조**를 설정합니다.
- 예를 들어, 네비게이션 바, 탭 바, 헤더 등이 여기에 포함될 수 있습니다.
- 하위 경로에 해당하는 화면들을 렌더링하는 **컨테이너 역할**을 합니다.
- `children` 프로퍼티를 통해 특정 라우트의 컴포넌트를 감쌀 수 있습니다.
- 공통된 레이아웃을 상속하여 하위 경로의 화면들에서 중복되는 레이아웃 정의를 피할 수 있습니다.

다음과 같은 파일 구조가 있다고 가정해 봅시다:

```
app/
|-- _layout.tsx
|-- home.tsx
|-- about.tsx
|-- index.tsx
```

- **_layout.tsx**: 공통 레이아웃 파일로, `home.tsx`와 `about.tsx`를 감싸는 역할을 합니다.
- **home.tsx**: `/home` URL에 해당하는 컴포넌트 파일입니다.
- **about.tsx**: `/about` URL에 해당하는 컴포넌트 파일입니다.
- **index.tsx**: `/index` URL에 해당하는 컴포넌트 파일입니다.

이 구조에서 `_layout.tsx` 파일은 `home.tsx`와 `about.tsx` 파일을 감싸고, 두 URL 모두에서 공통적으로 렌더링되는 헤더를 정의할 수 있습니다.

##### 🤔 근데 URL 기반으로 접속할 경우가 있나? (GPT)

React Native 앱에서 URL 기반 라우팅은 웹과 다르게 사용하는 경우가 제한적이긴 하지만, 특정 상황에서는 활용됩니다. **Expo Router**는 Next.js 스타일의 파일 기반 라우팅을 모바일 앱 환경에 도입했으며, URL 기반 라우팅을 지원합니다. 이를 이해하려면, 다음 두 가지 상황으로 나누어 생각해볼 수 있습니다.

##### **사용 사례**
1. **딥 링크(Deep Linking)**:
    - 사용자가 앱 외부(예: 이메일, 문자 메시지, 푸시 알림)에서 특정 화면으로 바로 이동할 때 URL이 사용됩니다.
    - 예: `myapp://profile/123` → 특정 사용자 프로필 화면으로 이동.
2. **앱 내 상태 공유**:
    - 특정 화면 상태를 URL로 공유하여 앱 내에서 직접 접근할 수 있도록 설정.
    - 예: `myapp://feed?filter=popular` → 인기 피드 필터를 적용한 화면.
3. **웹과의 통합(PWA)**:
    - React Native 앱이 웹(PWA)와 통합되는 경우, 동일한 URL 라우팅 방식으로 화면을 처리.
    - 예: `https://myapp.com/profile/123`와 `myapp://profile/123`가 동일한 화면을 렌더링.

##### 네비게이션 라우팅 뿐만 아니라 URL 라우팅이 들어온 이유 

1. **파일 기반 라우팅의 직관성**:
    - 파일 구조와 경로가 직접적으로 매핑되어 코드 가독성이 높아짐.
    - 예: `/app/profile/index.tsx` → `/profile`.
2. **딥 링크 및 상태 관리 통합**:
    - URL과 화면 상태를 매핑하여 앱 외부와의 통신 및 연동을 쉽게 만듦.
3. **웹과의 일관성**:
    - React Native 앱과 웹 앱을 동일한 라우팅 시스템으로 통합할 수 있음.
4. **라우팅 제어**:
    - 네비게이션과 URL을 함께 사용할 수 있어, 복잡한 앱 구조에서도 유연한 제어 가능.

##### 패키지 (tabs) 는 무엇을 의미할까? <Tab.Screen> 하지 않아도 작동되는거 같은데

Expo Router에서 사용하는 **`Tabs`** 패키지는 **탭 기반 네비게이션을 위한 컨테이너** 역할을 하며, `react-navigation`의 `createBottomTabNavigator`를 내부적으로 활용합니다. 이 패키지는 라우팅 구조와 자동 화면 매핑 기능을 제공하며, `<Tabs.Screen>`을 정의하지 않아도 파일 이름과 경로에 따라 화면을 자동으로 렌더링할 수 있습니다.

## 3. React Query를 이용해서 백엔드통신

크게 필요한 작업은 다음 세 가지로 말할 수 있습니다.

1. Query Client 정의 
2. 별도의 비동기 함수 정의 `ex) fetchPosts`
3. useQuery를 통한 훅 정의 

#### QueryClient 조금 더 자세하게 알아보기 

`QueryClient`는 **React Query의 핵심 객체**로, 다음 역할을 합니다:
    - **데이터 캐싱**: 요청한 데이터를 캐싱하고, 동일한 데이터를 다시 요청할 때 캐싱된 데이터를 활용.
    - **상태 관리**: 요청 상태(로딩 중, 성공, 실패)를 관리.
    - **전역 설정**: React Query의 기본 동작(예: 캐싱 시간, 재시도 횟수 등)을 설정.

> ★ QueryClient는 전역적으로 한 번만 선언되는 것이 맞습니다. 

`QueryClient`는 **데이터를 캐싱**하고 `queryKey`를 기준으로 요청 상태와 데이터를 관리합니다.

애플리케이션 전체에서 동일한 데이터를 여러 컴포넌트에서 사용하는 경우, 단일 `QueryClient`를 사용해야 데이터가 캐싱되고 공유됩니다. 만약 여러 개의 `QueryClient`를 사용하면 각 `QueryClient`가 독립적으로 데이터를 관리하므로, 동일한 데이터를 다시 요청하게 되어 불필요한 네트워크 요청이 발생합니다.

> ★ 보통 최상위 컴포넌트에서 **`QueryClientProvider`로 애플리케이션 전체를 감싸고**, 생성한 `queryClient`를 `client`라는 이름의 **prop**으로 넘겨줍니다

예제 코드는 다음과 같습니다. 
```javascript
import React from 'react';
import { QueryClient, QueryClientProvider } from 'react-query';
import Posts from './Posts';

const queryClient = new QueryClient(); 

export default function App() {
  return (
    <QueryClientProvider client={queryClient}> 
      <Posts />
    </QueryClientProvider>
  );
}

```

여기서 Context 라는 개념이 발생합니다. 
앞서 `<QueryClientProvider>` 를 사용해서 `queryClient`를 넘겨주게 되면 해당 객체는 `Context`에 보관됩니다. 즉, 계속해서 자식 컴포넌트에 queryClient를 넘겨주지 않고 `useQuery` 같은 훅을 사용할 수 있습니다. 

만약에 Context가 없다면 어떨까요? 계속해서 `QueryClient`를 파라미터로 넘겨주어야 합니다. 만약 계층 구조가 깊다면 `prop drilling` 문제가 발생할 수 있습니다. 
```javascript
const queryClient = new QueryClient();

function App() {
  return <Parent queryClient={queryClient} />;
}

function Parent({ queryClient }) {
  return <Child queryClient={queryClient} />;
}

function Child({ queryClient }) {
  const posts = queryClient.getQueryData(['posts']); // queryClient 직접 사용
  return <div>{posts ? posts.map(post => post.title) : 'No data'}</div>;
}
```

#### 별도의 비동기 함수 정의하기 

React Query에서 백엔드 통신을 처리하기 위해 **비동기 함수**를 정의합니다. 이 함수는 API와 통신하는 로직을 캡슐화하며, React Query의 `queryFn`이나 `mutationFn`으로 사용됩니다.

##### **역할**
- API와의 통신(데이터 가져오기, 생성, 수정, 삭제 등).
- 데이터를 변환하거나 전처리(필요한 경우).
- `axios`나 `fetch`를 사용해 비동기 요청을 처리.

##### **1. 기본 구조**
비동기 함수는 일반적으로 `async/await`를 사용하여 작성하며, 데이터를 반환합니다.

##### **GET 요청 예제**
```javascript
import axios from 'axios';

const fetchPosts = async () => {
  const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
  return response.data; // 서버에서 반환된 데이터를 그대로 반환
};
```

##### **POST 요청 예제**
```javascript
const createPost = async (newPost) => {
  const response = await axios.post('https://jsonplaceholder.typicode.com/posts', newPost);
  return response.data; // 생성된 데이터 반환
};
```

##### **2. 데이터 전처리**

API에서 받은 데이터를 바로 사용할 수 없는 경우, 비동기 함수에서 데이터를 가공하거나 변환할 수 있습니다.

```javascript
const fetchProcessedPosts = async () => {
  const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
  // 서버에서 받은 데이터를 가공
  return response.data.map((post) => ({
    id: post.id,
    title: post.title.toUpperCase(), // 제목을 대문자로 변환
    summary: post.body.slice(0, 50), // 본문 요약
  }));
};
```

##### **3. 재사용 가능한 비동기 함수 작성**

React Query에서 비동기 함수를 재사용 가능하도록 작성하면, 코드의 중복을 줄이고 유지보수를 쉽게 할 수 있습니다.

```javascript
const fetchData = async (endpoint) => {
  const response = await axios.get(`https://jsonplaceholder.typicode.com/${endpoint}`);
  return response.data;
};

// 다양한 엔드포인트에서 재사용
const fetchPosts = () => fetchData('posts');
const fetchComments = () => fetchData('comments');
```

##### **4. 에러 처리**

에러 처리는 React Query의 내부 로직으로 처리되지만, 비동기 함수에서 명시적으로 에러를 처리할 수도 있습니다.

```javascript
const fetchPosts = async () => {
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
    return response.data;
  } catch (error) {
    throw new Error('Failed to fetch posts'); // 에러 메시지를 React Query로 전달
  }
};
```
##### **5. 비동기 함수와 React Query의 연결**

React Query의 `useQuery`나 `useMutation`에서 비동기 함수를 `queryFn`이나 `mutationFn`으로 사용합니다.

`useQuery`에서 사용
```javascript
import { useQuery } from 'react-query';

const { data, isLoading, error } = useQuery(['posts'], fetchPosts);
```

`useMutation`에서 사용
```javascript
import { useMutation } from 'react-query';

const mutation = useMutation(createPost);

mutation.mutate({
  title: 'New Post',
  body: 'This is the body of the new post',
});
```

**비동기 함수 작성의 기본 원칙**:    
    1. API 호출을 처리하고 데이터를 반환.
    2. 필요하면 데이터를 가공하거나 전처리.
    3. 재사용 가능하도록 작성.
    4. 에러를 명시적으로 처리.

**React Query와 비동기 함수의 연결**:    
- `queryFn`이나 `mutationFn`으로 전달하여 API 요청을 React Query 훅에서 처리.
- React Query가 데이터 상태와 요청 상태를 자동으로 관리.

최종 예제는 다음과 같습니다.

```javascript
import axios from 'axios';

// 비동기 함수 정의
const fetchPosts = async () => {
  const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
  return response.data;
};

// React Query 훅에서 사용
import { useQuery } from 'react-query';

function Posts() {
  const { data, isLoading, error } = useQuery(['posts'], fetchPosts);

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```