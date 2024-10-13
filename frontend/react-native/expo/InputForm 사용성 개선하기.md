## 1. 개요
보통 회원 가입 시 아이디와 비밀번호 등을 입력 받습니다. 만약 아이디를 입력하기 위해 입력 폼을 클릭하고 입력이 끝난 뒤에는 다시 비밀번호 칸을 클릭하면 사용성이 떨어집니다.
입력이 끝난 후 바로 다음 칸으로 이동할 수 있도록 사용성을 개선해보겠습니다.

## 2. 사용성 개선하기 
기존의 코드는 다음과 같습니다.
```typescript
<InputField  
  caption={'아이디'}  
  placeholder={'teamfolio'}  
  error={login.errors.id}  
  touched={login.touched.id}  
  {...login.getTextInputProps('id')}  
/>
<InputField  
  caption={'닉네임'}  
  placeholder={'홍길동'}  
  error={login.errors.nickname}  
  touched={login.touched.nickname}  
  {...login.getTextInputProps('nickname')}  
/>
```

방법은 간단합니다. useRef 훅을 사용해서 컴포넌트를 참조하고 입력이 끝나면 해당 컴포넌트로 focus()를 옮겨줍니다.
```typescript
const nicknameRef = useRef<TextInput | null>(null);  
const dateRef = useRef<TextInput | null>(null);

<InputField  
  caption={'아이디'}  
  placeholder={'teamfolio'}  
  error={login.errors.id}  
  touched={login.touched.id}  
  returnKeyType="next"  
  blurOnSubmit={false}  
  onSubmitEditing={() => nicknameRef.current?.focus()}  
  {...login.getTextInputProps('id')}  
/>  
<InputField  
  ref = {nicknameRef}  
  caption={'닉네임'}  
  placeholder={'홍길동'}  
  error={login.errors.nickname}  
  touched={login.touched.nickname}  
  onSubmitEditing={() => dateRef.current?.focus()}  
  returnKeyType="next"  
  {...login.getTextInputProps('nickname')}  
/>
```

## 3. ForwardRef 
단, 문제가 있습니다. InputField 안에서 내부적으로도 ref를 사용하고 있습니다.
	InputField는 커스텀 컴포넌트로 `<TextInput>` 이외에도 오류 메세지를 표현하는`<Text>`등을 `<View>`로 묶고 있습니다. 
	InputField 내에서 `useRef`는 해당 `View`를 클릭하면 `TextInput`을 클릭한 것과 동일한 효과를 보여주기 위해서 `innerRef`를 사용하고 있습니다.

파라미터로 넘겨주는 ref와 내부적으로 정의한 innerRef를 같이 사용하는 방법을 알아보기 위해서는 `forwardRef`에 대해 알아야 합니다.
### 3-1. forwardRef란? 
공식 문서에 따르면 부모 컴포넌트가 자식 DOM에 접근할 때 forwardRef를 사용합니다.
하지만 위의 경우도 있는데, 자식 컴포넌트가 여러 개의 DOM 요소를 가지고 있을 때, 그리고 그 중 하나를 부모가 제어해야할 때도 forwardRef를 사용합니다. 

ForwardRef를 사용하면 기존의 코드를 변경해야 합니다.
```typescript
const InputField =  
  forwardRef(  
    ({disabled, caption, error, touched, onPress, ...props}: InputFieldProps,  
ref?:ForwardedRef<TextInput>) => {
              ...
}
```

이렇게 함수 형으로 만들어줍니다. 

또한, 유틸 함수를 사용해서 ref를 병합해줘야 합니다. 
```typescript
<TextInput  
  ref={ref? mergeRefs(innerRef,ref) : innerRef}
  ...
```

```typescript
function mergeRefs<T>(...refs: ForwardedRef<T>[]) {  
  return (node: T) => {  
    refs.forEach(ref => {  
      if (typeof ref === 'function') {  
        ref(node)  
      } else if (ref) {  
        ref.current = node  
      }  
    })  
  }  
}
```
### 주요 요소:

1. **`ForwardedRef<T>[]`**:
    - 이 함수는 가변 인자로 `ForwardedRef<T>` 타입의 배열을 받습니다. `ForwardedRef<T>`는 React에서 `ref`를 전달받을 때 사용되는 타입입니다. 이 타입은 함수형 `ref`(callback ref) 또는 `ref` 객체(`React.RefObject<T>`)가 될 수 있습니다.
    - 함수형 `ref`: `(instance: T | null) => void`
    - `ref` 객체: `{ current: T | null }`
      
2. **`node: T`**:
    - 이 `node`는 React의 컴포넌트에서 `ref`가 가리키는 DOM 요소나 컴포넌트 인스턴스입니다. 예를 들어, `TextInput`의 `ref`로 전달된 경우, `node`는 해당 `TextInput` DOM 요소가 됩니다.
      
3. **`refs.forEach(ref => {...})`**:
    - 이 부분은 전달받은 여러 `ref`에 대해 각각 처리합니다.
      
4. **`typeof ref === 'function'`**:
    - 만약 `ref`가 함수형 `ref`인 경우, `ref(node)`를 호출하여 `node`를 전달합니다. 이때 `node`는 해당 DOM 요소 또는 컴포넌트 인스턴스입니다.
      
5. **`ref.current = node`**:
    - 만약 `ref`가 `ref` 객체(`React.RefObject<T>`)인 경우, `ref.current`에 `node`를 할당합니다. 이를 통해 `ref` 객체가 가리키는 실제 DOM 요소나 컴포넌트 인스턴스가 `node`로 설정됩니다.