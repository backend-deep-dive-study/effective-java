# Raw 타입은 사용하지 말라

### 제네릭 클래스, 제네릭 인터페이스

- 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 것
- List 인터페이스 : 원소 타입을 나타내는 타입 매개변수 E를 받음
    - List<E> 로 표현

### 제네릭 타입

- 제네릭 타입은 **매개변수화 타입을** 정의
    - 클래스/인터페이스 이름 <타입 매개변수 나열>
    - List<String> : 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입
    - String : (정규 타입 E에 해당하는) 실제 타입 매개변수

### raw type

- 제네릭 타입에서 타입 매개변수를 사용하지 않는 경우
    - List<E>의 raw type = List
- 제네릭 타입을하나 정의하면 그에 딸린 raw type도 정의됨
- 타입 선언에서 제네릭 타입 정보가 지워진 것처럼 동작
    - 제네릭이 생기기 전 코드와 호환되도록 하기 위한 방법

### 제네릭 지원 전까지는..

```java
//Stamp 인스턴스만 취급한다
private final Collection stamps = ...;
```

- 주석으로만 명시함
- Stamp 대신 Coin을 넣어도 오류없이 컴파일되고 실행됨
- Coin을 다시 꺼내기 전까지는 오류를 알기 어려움
- 오류는 발생 즉시 / 컴파일할 때 발견하는 것이 좋은데 해당 코드는
    - 런타임에 오류를 발견할 수 있음
    - 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 떨어져 있을 가능성 큼
    - 발생한 예외에 해당 하는 부분의 코드를 직접 확인

### 매개변수화된 컬렉션 타입

```java
private final Collection<Stamp> stamps = ...;
```

- stamps에 Stamp 인스턴스만 넣어야 함을 컴파일러가 인지
    - Stamp 외 타입의 인스턴스를 넣으면 컴파일 오류 발생
    - 무엇이 잘못됐는지 정확히 알려줌
- 컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 형변환을 추가해둬서 실패하지 않음을 보장

### List vs List<Object>

- raw 타입을 쓰면 제네릭이 주는 안전성, 표현력 모두 잃음
- 자바에 제네릭이 도입되기 전 코드와의 호환성 때문에 놔둠
- List<Object>처럼 임의 객체를 허용하는 매개변수화 타입은 사용 가능
    - 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달

### 제네릭 하위 타입 규칙

- List<String>은 List<Object>의 하위 타입이 아님
    - List<String> : 문자열만 들어갈 수 있는 리스트
    - List<Object> : 어떤 타입의 객체든 들어갈 수 있는 리스트
- 제네릭 타입은 *공변성, *반공변성을 갖지 않아 이런 타입 간 관계 성립X
    - 제네릭 타입의 하위 타입 관계는 타입 간 상속 관계와 다르게 취급
- List<Object> 같은 매개변수화 타입을 사용할 때와 달리 List 같은 raw 타입을 사용하면 안전성을 잃음 → 타입을 지정하지 않아 어떤 객체든 넣을 수 있음 (런타임에 문제 발생)

```java
public static void main(String[] args) {
	List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);
}

private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
```

- 컴파일은 되지만 List(raw 타입)를 사용하여 경고 발생
- String s = strings.get(0); 의 결과를 형변화하려 할 때 ClassCastException 던딤
    - Integer를 String으로 변환 시도

```java
public static void main(String[] args) {
	List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);
}

private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
}
```

- 컴파일 불가

### 비한정 와일드카드 타입

- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때 사용
    - ? : 타입을 제한하지 않고 어떤 타입이든 허용
- 비한정 와일드카드 타입은 안전하고, raw 타입은 안전하지 않음
    - raw 타입 컬렉션에는 아무 원소나 넣을 수 있어 타입 불변식 훼손이 쉬움
    - 와일드카드의 경우 null 외 다른 원소를 넣으면 컴파일 에러 발생 → 컬렉션의 타입 불변성 훼손 막음
    

### raw 타입을 쓰지 말라는 규칙의 예외

- class 리터럴에는 raw 타입 사용
    - 자바 명세에서 class 리터럴에는 매개변수화 타입을 사용하지 못하게 함
- 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외 매개변수화 타입은 적용할 수 없음

### 정리

- raw 타입은 런타임 예외 발생 가능성이 있어 사용하면 안됨
- 현재 막아두지 않은 이유는 제네릭이 도입되기 전 코드와 호환성을 위해 제공

---

### 부록

- **공변성 (Covariance)**: 타입 파라미터가 상속 관계에 따라 자식 타입 → 부모 타입으로 변환되는 것을 의미
- **반공변성 (Contravariance)**: 부모 타입 → 자식 타입으로 변환되는 것을 의미
