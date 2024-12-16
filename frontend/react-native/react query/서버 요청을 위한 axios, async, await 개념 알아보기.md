## 1. 개요 
클라이언트에서 서버로 데이터를 주고받기 위해서는 HTTP 통신을 처리할 수 있는 방법이 필요합니다. 현대적인 브라우저 환경에서는 기본적으로 제공되는 `fetch` API를 사용할 수 있으며, 추가적인 기능과 편의성을 제공하는 라이브러리로 `axios`도 많이 사용하고 있습니다.

이번 글에서는 `axios`가 무엇인지 간단히 살펴보고, 이를 사용할 때 자주 함께 쓰이는 `async`와 `await` 키워드가 어떤 역할을 하며 왜 유용한지 알아보도록 하겠습니다.

## 2. axios란 
> Axios는 브라우저를 위해 Promise API를 활용하는 HTTP 비동기 통신 라이브러리입니다. 

Axios를 이해하기 위해선 비동기에 대해 제대로 된 이해가 필요합니다. 

#### 😎 동기와 비동기에 대해 알아보자
동기와 비동기의 차이는 **호출 되는 함수의 작업 여부를 신경쓰는지**에 따라 다릅니다. 

예를 들어, 동기의 경우 함수 A가 함수 B를 호출한 뒤, 함수 B의 반환 값을 계속 확인하면서 신경씁니다.

비동기의 경우 함수 A가 함수 B를 호출할 때 콜백 함수를 함께 전달하여, 함수 B 작업이 완료되면 해당 콜백 함수가 실행되도록 합니다.

#### Promise 객체에 대해 알아보자 
그렇다면 이제 Promise 객체에 대한 이해가 필요합니다. Axios는 HTTP 요청을 수행한 뒤, 그 결과를 Promise 형태로 반환합니다. 이를 통해 비동기적으로 서버에서 받은 응답을 처리할 수 있습니다. 

예를 들어, axios.get(), axios.post()를 호출하면 서버에서 HTTP 요청을 보내고 응답을 받는 즉시 그 결과를 Promise로 감싸서 반환합니다. 

참고로 Promise는 세 가지 주요 상태를 가집니다. 
1. 대기 (Pending)
	Promise가 생성되어, 아직 결과를 반환하지 않고(해결되지 않고) 기다리는 상태입니다.
2. 이행 (fulfilled)
	비동기 작업이 성공적으로 완료되어 값을 반환한 상태입니다.  
    `.then()` 블록이 여기서 실행됩니다.
3. 거부 (Rejected)
	비동기 작업이 실패하여 에러를 반환한 상태입니다.  
    `.catch()` 블록이 여기서 실행됩니다.

`axios`로부터 받은 `Promise` 객체를 통해 `.then()`, `.catch()`를 이용한 **체이닝**이 가능합니다. 이는 서버 응답 이후에 수행해야 할 후속 작업(데이터 가공, 화면 렌더링, 에러 처리 등)을 깔끔하게 연결할 수 있게 해줍니다. 

```js
axios.get('/api/data')
  .then(response => {
    console.log(response.data);
    // 응답 데이터를 이용한 후속 처리
  })
  .catch(error => {
    console.error(error);
    // 에러 처리
  });

```

그렇다면 이제 async, await를 사용함으로써 얻을 수 있는 장점에 대해 알아봅시다. 

만약 아래와 같이 첫 번째 조회 결과로 부터 데이터를 얻어, 두 번째 요청에 사용한다면 메소드 체이닝 형태로 다음과 같이 구현해야 합니다. 
```js
const test = () {
	axios.get('/api/data')
		.then((response) => {
			return axios.get('/api/users/' + response.data.userId)
				.then((response) => {
					console.log("Query user : ", response.data)
				})
				.catach(()=>{})
		})
		.catch((error) => {
			console.log("Error", error)
		})
}
```

이를 다음과 같이 간소화 할 수 있습니다.
```js
const test = async () {
	try {
		const first = await axios.get('/api/users/')
		const userId = first.data.userId;

		const second = await axios.get('/api/users/' + userId)
		console.log("response : ", second.data)
	} catch (error) {
		console.log("error : ", error)
	}
}
```

즉, `async/await`를 사용하면 `then()` 체인을 깔끔하게 동기 흐름처럼 표현할 수 있으며, 에러 처리도 `try...catch` 문으로 직관적으로 처리할 수 있습니다.

#### 🤔 어차피 데이터를 다 받고 실행되어야 하면 처음부터 동기로 구현하는게 좋지 않나?

실제 웹 환경에서는 해당 요청의 응답을 기다리는 동안에도 해야 할 일이 많습니다. 화면 갱신, 다른 API 호출, 사용자 입력 처리 등은 응답을 기다리는 것과는 별개로 동시에(또는 순서에 구애받지 않고) 진행될 수 있습니다. 바로 이러한 이유로 비동기 처리가 필요하며, `async/await`는 이 비동기 로직을 동기적 흐름처럼 읽기 쉽게 표현하는 데 도움이 되는 문법적 도구인 것입니다.

## Reference
- https://f-lab.kr/insight/async-programming-promise-async-await-20240521
- https://third9.github.io/posts/Axios%EB%8B%A4%EC%96%91%ED%95%98%EA%B2%8C_%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0-async_await%EC%82%AC%EC%9A%A9/