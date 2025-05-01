# 지연 초기화는 신중히 사용하라

### 지연 초기화 (Lazy Initialization)
- 필드의 초기화 시점을 사용 시점으로 미루는 기법
- 목적: 클래스의 인스턴스를 생성하는 데 드는 비용 최적화, 클래스 초기화 순환 문제를 피하기 위해 사용됨
- 하지만 무분별하게 사용하면 성능이 저하될 수 있음

### 지연 초기화가 필요한 때
- 초기화 비용이 크고 사용 빈도는 낮은 경우
- 지연 초기화 전후의 성능을 측정해보는 것이 좋음

### 구현 방법
1. 일반적인 필드 초기화
```java
private final FieldType field = computeFieldValue();
```
- 단순하고 명확함
- 대부분의 경우 이 방법이 가장 좋음

2. 지연 초기화 - 정적 필드용 (holder 클래스 사용)
```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

public static FieldType getField() {
    return FieldHolder.field;
}
```
- JVM의 클래스 초기화 시점 보장: 스레드 안전하면서도 성능 좋음
- 정적 필드에만 사용 가능

3. 지연 초기화 - 인스턴스 필드용 이중검사 관용구(double-check idiom)
```java
private volatile FieldType field;

public FieldType getField() {
    FieldType result = field;
    if (result == null) {
        synchronized(this) {
            result = field;
            if (result == null)
                field = result = computeFieldValue();
        }
    }
    return result;
}
```
- 한 번은 동기화 없이 검사, 두 번째는 동기화하여 검사
- volatile 키워드가 필요
- 반복해서 초기화해도 상관없는 인스턴스 필드일 경우 이중검사에서 두 번째 검사 생략 가능

4. 단일검사(racy single-check) 관용구
```java
private FieldType field;

public FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```
- 모든 스레드가 필드 값을 다시 계산해도 상관없고 필드 타입이 long, double을 제외한 다른 기본 타입인 경우
- volatile 한정자를 없애도 됨


### 결론
- 지연 초기화는 필요할 때만, 정당한 이유가 있을 때만 사용하라
- 대부분의 경우는 일반적인 초기화 방식이 가장 좋다
