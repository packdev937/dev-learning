Map은 Key : Value 쌍으로 구성된 자료구조입니다. 즉, 정렬을 할 때는 Key를 기준으로 정렬할 수도, Value를 기준으로 정렬할 수도 있습니다. 

정렬 할 때는 `keySet()`을 이용합니다. 
Map의 경우 Key를 이용하여 값을 얻기 때문에 KeySet을 어떻게 정렬하는가에 따라 정렬 기준을 다르게 정의할 수 있습니다.

### Key 값으로 정렬
```java
List<String> keys = new ArrayList<>(map.keySet());

Collections.sort(keys);

for (String key : keys) {
    System.out.print("Key : " + key);
    System.out.println(", Value : " + map.get(key));
}
```

### Value 값으로 정렬
```java
List<String> keys = new ArrayList<>(map.keySet());

keys.sort((o1, o2) -> map.get(o2).compareTo(map.get(o1)));

for (String key : keys) {
    System.out.print("Key : " + key);
    System.out.println(", Val : " + map.get(key));
}
```

