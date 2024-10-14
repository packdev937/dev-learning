## 1. 개요

이번 포스팅에서는 TypeScript를 사용하여 React 또는 React Native에서 Props를 정의하는 방법에 대해 알아보겠습니다. `interface`와 `type`을 활용해 컴포넌트의 props 타입을 선언하고, 각 방식의 차이점을 비교해 보겠습니다.

## 2. Props 정의하기
### 2-1. interface

TypeScript에서 React 컴포넌트의 `props`를 정의할 때, 주로 `interface`를 사용합니다. interface는 컴포넌트가 받아야 하는 props의 구조를 명확하게 정의하고, 컴파일 타임에 타입 검사를 수행할 수 있게 해줍니다.

```typescript
interface CustomButtonProps {
  label: string;
  variant?: "filled" | "outlined";
  size?: "medium" | "large";
}
```

- `?`는 해당 속성이 선택적임을 나타냅니다.  
- `CustomButtonProps` 인터페이스는 label 속성을 필수로 받고, variant와 size는 선택적 속성으로 정의합니다.

### 2-2. type
`interface` 대신 `type`을 사용하여 props를 정의할 수도 있습니다. `type`은 더 유연하게 타입을 선언할 수 있으며, 다양한 데이터 타입을 정의하는 데 유용합니다.

```typescript
type CustomButtonProps = {
  label: string;
  variant?: "filled" | "outlined";
  size?: "medium" | "large";
};
```

### 2-3. interface와 type의 차이점

**interface**:
- 주로 **객체의 구조**를 정의하는 데 사용되며, 다른 **`interface`**와 **확장**이 용이합니다.
- 확장성과 선언 병합이 가능해, **큰 프로젝트**에서 주로 사용됩니다.

```typescript
interface BaseButtonProps {
  label: string;
}

interface CustomButtonProps extends BaseButtonProps {
  variant?: "filled" | "outlined";
  size?: "medium" | "large";
}
```

**type**:
- **유연성**이 높아, 기본 타입 (예: 유니온 타입, 튜플 등)을 정의할 때 자주 사용됩니다.
- **객체 타입 외에도** 다양한 형태의 타입을 정의할 수 있습니다.

```typescript
type ButtonVariant = "filled" | "outlined";
type ButtonSize = "medium" | "large";

type CustomButtonProps = {
  label: string;
  variant?: ButtonVariant;
  size?: ButtonSize;
};
```

## 3.예제로 알아보기

첫 번째 예제로 `CustomButtonProps` 인터페이스를 사용해 컴포넌트의 props 타입을 선언한 후, props의 값을 기본값으로 설정합니다.

```typescript
interface CustomButtonProps {
  label: string;
  variant?: "filled" | "outlined";
  size?: "medium" | "large";
}

function CustomButton({
  label,
  variant = 'filled',
  size = 'large',
}: CustomButtonProps) {
  // CustomButton 컴포넌트 구현
}
```

위 코드에서, `variant`와 `size`에 기본값을 할당하여, props가 전달되지 않더라도 **기본 동작을 보장합니다.

다음 예제입니다. 

React Navigation에서는 **네비게이션과 라우팅 관련 props**를 정의할 때 **유틸리티 타입**을 자주 사용합니다. 예를 들어, `StackScreenProps`를 사용하여 스택 네비게이터에서 화면의 `props`를 정의할 수 있습니다.

```typescript
type AuthHomeScreenProps = StackScreenProps<
  AuthStackParamList,
  typeof authNavigation.AUTH_HOME
>;
```

- **`StackScreenProps`**: React Navigation에서 제공하는 유틸리티 타입으로, 스택 네비게이터에서 각 화면이 **navigation**과 **route**라는 두 가지 주요 속성을 가지도록 강제합니다.
  - **`navigation`**: 화면 간의 이동과 관련된 작업을 수행하는 객체.
  - **`route`**: 현재 화면에 전달된 파라미터에 접근할 수 있는 객체.

```typescript
const authNavigation = {
  AUTH_HOME: 'AUTH_HOME',
  AUTH_LOGIN: 'AUTH_LOGIN',
  AUTH_REGISTER: 'AUTH_REGISTER',
} as const;

type AuthStackParamList = {
  AUTH_HOME: undefined;
  AUTH_LOGIN: { redirectUrl?: string };
  AUTH_REGISTER: { referrerId?: string };
};
```

위 코드에서 `AuthStackParamList`는 각 화면의 경로와 파라미터를 정의합니다. `AuthHomeScreenProps`는 이 정보를 기반으로 특정 화면에 전달되는 props 타입을 설정합니다.
