JavaScript에서 `Set` 객체는 중복 값을 저장하지 않는 컬렉션입니다. 그러나 객체의 중복을 판단하는 기준은 객체의 내용이 아닌, 객체의 메모리 주소입니다. 따라서 같은 내용을 가진 다른 객체는 `Set`에 중복으로 간주되지 않고 추가될 수 있습니다.

### `Set`의 동작 방식

#### 1. 객체의 중복 판단 기준
`Set`은 객체의 내용을 비교하지 않고, 객체의 메모리 주소(참조)를 기준으로 중복을 판단합니다. 같은 내용을 가진 객체라도 서로 다른 메모리 주소를 가지면 `Set`은 이를 중복으로 간주하지 않습니다.

```javascript
const obj1 = { name: 'apple', price: 9 };
const obj2 = { name: 'banana', price: 5 };
const objs = new Set([obj1, obj2]);

console.log(objs); // Set { { name: 'apple', price: 9 }, { name: 'banana', price: 5 } }

obj1.price = 10;
objs.add(obj1);

console.log(objs); // Set { { name: 'apple', price: 10 }, { name: 'banana', price: 5 } }
```

위 예제에서 `obj1`의 `price`를 변경하고 `Set`에 다시 추가해도 중복으로 간주되지 않습니다. 이는 `Set`이 객체의 참조(메모리 주소)를 기준으로 중복을 판단하기 때문입니다.

#### 2. 새로운 객체 추가
새로운 객체를 `Set`에 추가하면, 비록 내용이 동일하더라도 참조가 다르기 때문에 중복으로 간주되지 않습니다.

```javascript
const obj3 = { name: 'banana', price: 5 };
objs.add(obj3);

console.log(objs); // Set { { name: 'apple', price: 10 }, { name: 'banana', price: 5 }, { name: 'banana', price: 5 } }
```

### 객체 비교 방법

Java의 `equals`와 `hashCode` 메서드와 유사한 방식으로 객체를 비교하기 위해서는 객체의 내용을 직접 비교해야 합니다. JavaScript에서는 객체의 내용을 비교하기 위해 별도의 함수나 라이브러리를 사용할 수 있습니다.

예를 들어, 객체의 내용을 비교하여 중복 여부를 판단하는 함수를 작성할 수 있습니다.

```javascript
function isEqual(obj1, obj2) {
  return obj1.name === obj2.name && obj1.price === obj2.price;
}

function addToSet(set, obj) {
  for (let item of set) {
    if (isEqual(item, obj)) {
      return; // 이미 존재하는 객체
    }
  }
  set.add(obj); // 새로운 객체 추가
}

const customSet = new Set();
addToSet(customSet, { name: 'apple', price: 9 });
addToSet(customSet, { name: 'banana', price: 5 });
addToSet(customSet, { name: 'banana', price: 5 });

console.log(customSet); // Set { { name: 'apple', price: 9 }, { name: 'banana', price: 5 } }
```

### 타입 제한 방법

JavaScript는 정적 타입을 지원하지 않기 때문에 Java처럼 `HashMap<String>`과 같은 타입 제한을 직접적으로 할 수는 없습니다. 그러나 TypeScript를 사용하면 타입을 지정하여 객체를 사용할 수 있습니다.

#### TypeScript 예제

```typescript
interface Fruit {
  name: string;
  price: number;
}

const fruitSet: Set<Fruit> = new Set();

const apple: Fruit = { name: 'apple', price: 9 };
const banana: Fruit = { name: 'banana', price: 5 };

fruitSet.add(apple);
fruitSet.add(banana);

console.log(fruitSet); // Set { { name: 'apple', price: 9 }, { name: 'banana', price: 5 } }

// 같은 내용을 가진 새로운 객체
const anotherBanana: Fruit = { name: 'banana', price: 5 };
fruitSet.add(anotherBanana);

console.log(fruitSet); // Set { { name: 'apple', price: 9 }, { name: 'banana', price: 5 }, { name: 'banana', price: 5 } }
```

TypeScript를 사용하면 객체의 타입을 지정할 수 있으며, 이를 통해 타입 안전성을 확보할 수 있습니다.

### 결론

- JavaScript의 `Set`은 객체의 내용을 비교하지 않고 참조(메모리 주소)를 기준으로 중복을 판단합니다.
- 객체의 내용을 비교하여 중복을 판단하려면 커스텀 함수를 작성해야 합니다.
- TypeScript를 사용하면 객체의 타입을 지정하여 타입 안전성을 확보할 수 있습니다.

이러한 방법들을 통해 객체의 중복을 판단하고 타입을 제한할 수 있습니다.