## 1. 개요
여러 컴포넌트에서 재 사용할 수 있는 로직인 Custom Hook에 대해 조금 더 자세히 알아보겠습니다. 
```javascript
import { useEffect, useState } from 'react'  
  
interface UseFormProps<T> {  
  initialValue: T  
  validate: (values: T) => Record<keyof T, string>  
}  
  
function useForm<T>({ initialValue, validate }: UseFormProps<T>) {  
  const [values, setValues] = useState(initialValue)  
  const [touched, setTouched] = useState<Record<string, boolean>>({})  
  const [errors, setErrors] = useState<Record<string, string>>({})  
  
  const handleChangeText = (name: keyof T, text: string) => {  
    setValues({  
      ...values,  
      [name]: text,  
    })  
  }  
  
  const handleBlur = (name: keyof T) => {  
    setTouched({  
      ...touched,  
      [name]: true,  
    })  
  }  
  
  const getTextInputProps = (name: keyof T) => {  
    const value = values[name]  
    const onChangeText = (text: string) => handleChangeText(name, text)  
    const onBlur = () => handleBlur(name)  
  
    return { value, onChangeText, onBlur }  
  }  
  
  useEffect(() => {  
    const newErrors = validate(values)  
    setErrors(newErrors)  
  }, [validate, values])  
  
  return { values, errors, touched, getTextInputProps }  
}  
  
export default useForm
```

예제는 다음과 같습니다. 우선 하나씩 뜯어가며 비교해보겠습니다. 
## 2. Props 
Props는 interface와 type 두 키워드를 사용해서 정의 가능 합니다. 
`interface`
``` javascript
interface UseFormProps<T> {
  initialValue: T;
  validate: (values: T) => Record<keyof T, string>;
}
```

`type`
```javascript
type UseFormProps<T> = {
  initialValue: T;
  validate: (values: T) => Record<keyof T, string>;
};
```

Props는 쉽게 말해 매개 변수라고 생각하면 됩니다. 
```javascript
type UseFormProps<T> = { 
	initialValue: T; 
	validate: (values: T) => Record<keyof T, string>; 
}; // useForm 훅 정의 

function useForm<T>({ initialValue, validate }: UseFormProps<T>) {
```

보시면 type 내에 있는 필드 들이 그대로 매개 변수에 사용 되는 것을 볼 수 있습니다. 
### ?. Record란 
Record는 Java의 Map과 비슷합니다. 둘 다 key-value가 존재합니다.
다만 다른 점은 Record는 정적인 객체 타입이며, Map은 동적인 객체 타입으로 런타임 중에 값을 추가할 수 있습니다. 

둘의 생김새는 다음과 같습니다. 
```
// Java
Map<String, String> map = new HashMap<>();

// JavaScript
type Person = {
  name: string;
  age: number;
};

type ErrorMessages = Record<keyof Person, string>;

const errors: ErrorMessages = {
  name: "Name is required.",
  age: "Age must be a positive number."
};
```

하지만 유의해야 할 점은 다음과 같습니다. 
Record의 key가 Person이라고 해서 Person 객체를 넣는 것이 아닌 Person 내 필드 값을 키로 가진다는 뜻입니다.

🤔 그렇다면  `validate: (values: T) => Record<keyof T, string>` 이 형태는 무엇일까?
쉽게 말해 요청 값과 응답 값이 다른 것입니다. 