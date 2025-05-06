# readObject 메서드는 방어적으로 작성하라

## readObject()란
- readObject(ObjectInputStream in)은 Serializable을 구현한 클래스에서 역직렬화 중 자동으로 호출되는 메서드
- 객체를 바이트 스트림에서 복원할 때 호출되어, 객체의 필드를 설정하거나 추가 작업을 수행하는 데 사용됨
- 즉, readObject()는 생성자를 대체하는 비공식 생성자

## 위험한 이유
### 1. 생성자를 우회함
- 직렬화/역직렬화는 일반 생성자가 호출되지 않음
- 불변식(invariant)을 검사하거나 보장하는 로직이 무시

### 2. 외부 입력 기반
- 역직렬화는 외부에서 보낸 바이트 스트림을 그대로 해석하여 악의적이거나 잘못된 데이터를 포함할 수 있음
- 역직렬화 공격으로 불변 필드를 위반하거나, 객체 간 관계를 위조할 수 있음

## 해결 방법
### 1. 불변성(invariant) 체크
객체가 생성될 때 반드시 만족해야 하는 조건들(불변식)을 점검

```java
if (start.compareTo(end) > 0) {
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

### 2. 방어적 복사(Defensive Copy)
가변 객체를 역직렬화할 경우, 외부에서 전달된 참조를 직접 사용하면 불변성, 캡슐화가 깨짐

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    // 방어적 복사
    start = new Date(start.getTime()); 
    end = new Data(end.getTime());
}
```
- Date, List, 배열 등 가변 객체는 반드시 복사하여 사용
- final 필드는 방어적 복사가 불가능함
- 객체를 역직렬화할 때 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.

### 4. ObjectInputValidation
ObjectInputStream.registerValidation()을 사용하면 readObject() 이후 후처리 검증을 수행할 수 있음

```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject();
    in.registerValidation(() -> {
        if (this.name == null || this.name.isEmpty()) {
            throw new InvalidObjectException("name cannot be null or empty");
        }
    }, 0);
}
```

## 결론 : 안전한 readObject 메서드 작성 지침
1. 방어적 복사 : private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라.
2. 불변식 검사 : 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다.
3. ObjecetInputValidation : 역직렬화 후 객체 그래프 전체의 유효성을 검사할 때
4. 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.
