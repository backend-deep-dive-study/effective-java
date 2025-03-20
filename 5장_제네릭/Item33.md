# 타입 안전 이종 컨테이너를 고려하라

### 타입 안전 이종 컨테이너 패턴

- 제네릭에서 매개변수화되는 대상은 컨테이너 자신
  - 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한됨
  - ex) `Set<T>`는 하나, `Map<K, V>`는 키와 값 2개로 제한
- 컨테이너 대신 키를 매개변수화하는 방법
- 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공

### 타입 토큰

- 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴
- `Class<?>` 객체나 직접 구현한 키 타입을 키로 사용 가능

### Favorites 예시

```java
  public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
      favorites.put(Objects.requireNonNull(type), instance);
    };
    public <T> T getFavorite(Class<T> type) {
      return type.cast(favorites.get(type));
    };
  }
```

- 타입 안전한 컨테이너
  - 여러 가지 타입의 원소를 담을 수 있음
  - `Map<Class<?>, Object>`
- 키와 값의 관계
  - `Class<?>` : 키는 와일드카드 타입이 중첩됨 -> 모든 키가 서로 다른 매개변수화 타입일 수 있음 (다양한 타입 지원)
  - `Object` : 모든 값이 키로 명시한 타입임을 보증하지는 않음
- `putFavorite` : Class 객체와 인스턴스를 추가해 관계 지으면 끝
- `getFavorite` : 객체의 타입은 Object지만 T로 바꿔 반환해야 함
  - Class의 cast 메서드를 사용해 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환 해야 함
  - `cast` : 주어진 인수가 Class 객체가 알려주는 타입인지 검사하고 맞으면 그대로 반환, 아니면 ClassCastException
  - Class 클래스가 제네릭이라는 이점을 활용해 타입 안전하게 만들어줌

### 타입 안전성이 깨질 수 있는 경우

- 악의적인 클라이언트가 로 타입(Raw Type)을 넘기는 경우
  - `Class<?>`의 로 타입을 사용하면 타입 안전성이 깨질 수 있음
  - 제네릭에서 로 타입 사용을 피해야 하는 이유 중 하나
- 실체화 불가 타입 문제
  - `List<String>.class` 같은 타입 정보를 런타임에 유지할 수 없음 → `List.class`만 존재함
  - 제네릭 타입 매개변수는 실체화되지 않음 (Type Erasure 때문)

### 한정적 타입 토큰

- 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰
- ex) 애너테이션 API
  ```java
    public <T extends Annotation>
      T getAnnotation(Class<T> annotationType);
  ```
  - 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 애너테이션을 반환, 아니면 Null 반환
  - 키가 애너테이션 타입인 타입 안전 이종 컨테이너

### `asSubclass`를 활용한 형변환 제한

- 특정 클래스 계층으로 변환할 때 사용
- `Class<?>`를 안전하게 특정 타입의 `Class<T>`로 변환 가능
- `asSubclass`(Number.class)는 numberClass가 Number의 서브타입이 아니면 ClassCastException을 발생시킴
- 이를 활용하면 타입 안전성을 더욱 강화 가능

### 결론
- 제네릭 컨테이너 대신 키(`Class<T>`)를 매개변수화하면 여러 타입을 안전하게 저장 가능
- `Map<Class<?>, Object>`를 활용하여 타입 안전 이종 컨테이너를 구현할 수 있음
- cast()를 사용하면 타입 변환을 안전하게 수행할 수 있음
- 로 타입을 사용하면 타입 안전성이 깨질 수 있으므로 주의 필요
- 실체화 불가 타입은 직접 저장할 수 없으며, 한정적 타입 토큰을 활용해야 함
- asSubclass를 활용하면 특정 계층 내에서만 형변환을 허용 가능
