## 1. 개요 
React Native를 통해서 앱 만드는 과정 중 `ref`이 자주 나왔습니다. 함수의 파라미터로 넘기기도 하고, 반환 값으로 선언하기도 하는 `ref`에 대해 조금 더 자세히 알 필요가 있다고 생각했고, 공식 문서를 보면서 학습한 내용을 다시 한번 복습해보도록 하겠습니다. 

## 2. Refs란
> When you want a component to “remember” some information, but you don’t want that information to [trigger new renders](https://react.dev/learn/render-and-commit), you can use a _ref_.

직역하자면 새로운 랜더링을 발생시키지 않으면서 어떤 정보를 기억하고 싶을 때 ref를 사용합니다. 

🥹어떤 정보를 기억할 때 새로운 랜더링이 발생한다는 건 어떤 상황일까요?

새로운 렌더링이 발생하는 상황은 다음과 같습니다.

React에서 컴포넌트가 상태(state)나 속성(props)을 변경할 때, 그 컴포넌트는 새로운 렌더링을 발생시킵니다. 이는 React가 변경된 데이터를 반영하기 위해 UI를 다시 그려야 하기 때문입니다. 이러한 재렌더링은 다음과 같은 상황에서 발생할 수 있습니다:

1. **상태 변경**: 컴포넌트의 상태가 변경될 때
```jsx
const [count, setCount] = useState(0);

const increment = () => {
  setCount(count + 1); // 상태가 변경되어 컴포넌트가 다시 렌더링됨
};

```

2. **속성 변경**: 부모 컴포넌트로부터 전달받은 속성이 변경될 때
```jsx
function ParentComponent() {
  const [value, setValue] = useState(0);

  return <ChildComponent value={value} />;
}

function ChildComponent({ value }) {
  // value가 변경되면 ChildComponent는 다시 렌더링됨
}
``` 

🤔 그렇다면 `Ref`는 이러한 상황에 어떻게 대처하는 걸까요? 
Ref는 컴포넌트가 특정 정보를 기억해야 하지만, 그 정보가 변경될 때 컴포넌트를 다시 렌더링할 필요가 없을 때 사용합니다. 예를 들어, 다음과 같은 상황에서 유용합니다.

특정 DOM 요소에 접근할 때 
```jsx
const inputRef = useRef(null);

const focusInput = () => {
  inputRef.current.focus(); // input 요소에 포커스를 줌
};

return <input ref={inputRef} />;
```

이전 상태를 기억해야 하지만, 그 값이 변경될 때 렌더링이 필요하지 않을 때 
```jsx
const renderCount = useRef(0);
renderCount.current += 1;

console.log(`렌더 횟수: ${renderCount.current}`);

```

## 3. DOM에 접근? 자세히 알아보자 
위에 예제에 나왔던 것 중 DOM에 접근할 때 사용한다고 했습니다. 앞에서 설명한 값을 기억하는 것과는 조금 거리가 먼 예제인데요, 해당 경우에도 동일하게 `Ref`를 사용합니다. 

해당 예제는 값을 변경하는 상황이 아니라, DOM 요소에 대한 참조를 유지하고 이를 통해 DOM 조작을 수행합니다.

```jsx
import React, { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus(); // input 요소에 포커스를 줌
  };

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

export default MyComponent;
```

- `useRef(null)`은 `inputRef`라는 변수를 생성하고, 초기값을 `null`로 설정합니다.
- `<input ref={inputRef} />`은 해당 input 요소에 `ref`를 설정하여, `inputRef.current`가 해당 DOM 요소를 참조하도록 합니다.
- `focusInput` 함수는 `inputRef.current.focus()`를 호출하여 해당 input 요소에 포커스를 줍니다.

이 과정에서 `inputRef`의 값이 변경되지만, 이는 컴포넌트의 상태나 속성이 아니므로 새로운 렌더링을 발생시키지 않습니다. Ref는 단순히 DOM 요소나 값을 기억하고, 필요할 때 접근할 수 있는 방법을 제공합니다.

## 4. useEffect와 useRef 
외부 API를 호출한다고 가정해봅시다. 매번 랜더링 할 때 마다 API를 호출하면 비효율적이니 우리는 useEffect를 통해 호출합니다. 근데 useRef 또한 충분히 사용될 여지가 보입니다. 

🤔 어떤 상황에 어떤 훅을 사용하는 것이 효과적일까요?
사실 useRef를 통한 외부 API를 사용할 때는 제약이 붙습니다. *"변경되지 않는"*
즉, 재랜더링 없이 특정 정보를 기억하거나 DOM 요소를 조작할 때 사용됩니다. 

예를 들어, 이전 객체 정보를 저장하고 있어야 되는 상황이라고 가정해보겠습니다.
```jsx
function PreviousValueComponent({ value }) {
  const previousValueRef = useRef();

  useEffect(() => {
    previousValueRef.current = value; // 현재 값을 이전 값으로 저장
  }, [value]);

  return (
    <div>
      <p>Current value: {value}</p>
      <p>Previous value: {previousValueRef.current}</p>
    </div>
  );
}
```

반면 위의 경우를 제외하고 대부분의 외부 API를 호출하는 경우에는 useEffect를 사용합니다. useEffect는 지정된 값이 바뀔 때 재 랜더링을 진행합니다. 매번 동일한 경우 랜더링을 진행하지 않지만 값이 바뀌면 해당 값은 업데이트 해야되기 때문입니다. 따라서 보다 더 적합한 경우라고 할 수 있습니다.  

## 5. Refs Challenge 
### 5-1. Send & Undo
메시지를 입력하고 "Send" 버튼을 클릭하세요. "Sent!" 알림이 나타나기 전까지 3초의 지연이 발생합니다. 이 지연 동안 "Undo" 버튼을 볼 수 있습니다. 이 "Undo" 버튼을 클릭하면 "Sent!" 메시지가 나타나는 것을 막아야 합니다. 이 버튼은 handleSend 동안 저장된 timeout ID에 대해 clearTimeout을 호출하여 이를 수행합니다. 그러나 "Undo"를 클릭해도 "Sent!" 메시지가 여전히 나타납니다. 왜 작동하지 않는지 찾아서 수정하세요.

기존의 코드는 다음과 같습니다. 
```jsx
import { useState } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  let timeoutID = null;

  function handleSend() {
    setIsSending(true);
    timeoutID = setTimeout(() => {
      alert('Sent!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutID);
  }

  return (
    <>
      <input
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        disabled={isSending}
        onClick={handleSend}>
        {isSending ? 'Sending...' : 'Send'}
      </button>
      {isSending &&
        <button onClick={handleUndo}>
          Undo
        </button>
      }
    </>
  );
}
```

Undo 버튼을 눌러도 여전히 Alert가 갔던 이유는 timeoutId가 let으로 선언되었기 때문에 `clearTimeout`이 제대로 동작하지 않기 때문입니다.

이를 해결하기 위해서는 `Ref`를 사용하여 참조를 유지하는 방법이 있습니다. 
```jsx
...
const timeoutId = useRef(null)

function handleSend(){
	setIsSending(true)
	timeoutId.current = setTimeout (() ...
}

function handleUndo() {
	setIsSending(false)
	clearTimeout(timeoutId.current)
}
```
### 5-2. On & Off
텍스트가 계속 off로 유지된다는 문제였습니다. `useRef`를 사용하여 상태를 저장하고 업데이트하면 `Ref`는 컴포넌트를 재랜더링 하지 않기 때문에 값이 바뀌지 않습니다. 따라서 `useState`를 사용해야 합니다. 

```jsx
import { useState, useRef } from 'react';

export default function Toggle() {
  const [isOn, setIsOn] = useState(true);
  
  return (
    <button onClick={() => {
      setIsOn(!isOn);
    }}>
      {isOn ? 'On' : 'Off'}
    </button>
  );
}
```

### 5-3. DebouncedButton
세 개의 버튼이 있고, 하나의 버튼이 눌리면 2초 뒤에 알람이 발행합니다. 다만, 2초가 되기 전에 다른 버튼을 누른다면 해당 작업은 취소되고 결국 제일 마지막에 누른 버튼의 알람이 발행됩니다.
```jsx
let timeoutID;

function DebouncedButton({ onClick, children }) {
  return (
    <button onClick={() => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```

문제가 발생했던 원인은 `timeoutId`를 전역으로 관리하고 있었기 때문입니다. 버튼을 누를 때 마다 값이 새로 갱신되니 알람이 제대로 발행되지 않습니다. 

해결책은 함수 내에서 `timeoutId`를 `Refs`로 관리하는 것입니다. 
```jsx
function DebouncedButton({ onClick, children }) {
  const timeoutIDRef = useRef(null);

  return (
    <button onClick={() => {
      clearTimeout(timeoutIDRef.current);
      timeoutIDRef.current = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}
```

### 5-4. Latest Change 
가장 최근의 변경 사항을 반영하고 싶은 문제입니다. 

`useEffect`를 사용해서 변경이 감지되면 해당 값을 변경해줍니다. 참고로 `useEffect`는 의존성 배열에 값이 바뀔 때 실행됩니다.
```jsx
import { useState, useRef, useEffect } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const textRef = useRef(text);

  useEffect(() => {
    textRef.current = text;
  }, [text]);

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + textRef.current);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}

```
## Reference
- https://react.dev/learn/referencing-values-with-refs#