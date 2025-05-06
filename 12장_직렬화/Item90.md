# 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

> Serializable을 구현하기로 결정한 순간 언어의 정상 메커니즘인 생성자이외의 방법으로 인스턴스를 생성할 수 있게 된다. <br>
> → 역직렬화(deserialization) 를 통해 생성자를 호출하지 않고도 객체를 생성할 수 있다.

## 직렬화 프록시 패턴 (serialization proxy pattern)



```java
public class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
    }

    // 1. 중첩 클래스: 직렬화 프록시
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private static final long serialVersionUID = 28370820129832193L;

        // 4. 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드
        private Object readResolve() {
            return new Period(start, end);
        }
    }

    // 2. 직렬화 프록시 패턴용 writeReplace 메서드
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // 3. 직렬화 프록시 패턴용 readobject 메서드
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
```

1. 중첩 클래스: 직렬화 프록시
   
    > 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언 <br>
    > 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시이다.
    
    - 중첩 클래스의 생성자는 단 하나여야 함
    - 바깥 클래스를 매개변수로 받아야 함
    - 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사
    - 일관성 검사, 방어적 복사 필요 없음
    - 설계상, 직렬화 프록시의 기본 직렬화 형태는 바깥 클래스의 직렬화 형태로 쓰기에 이상적
    - 바깥 클래스와 직렬화 프록시 모두 `Serializable`을 구현한다고 선언해야 함

2. 직렬화 프록시 패턴용 `writeReplace` 메서드

    > 이 메서드는 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 대신 `SerializationProxy의` 인스턴스를 반환하게 하는 역할을 한다. <br>
    > 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다. <br>
    > `writeReplace` 덕분에 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다.

3. 직렬화 프록시 패턴용 `readobject` 메서드

    > 하지만 공격자가 클래스의 불변식을 깨뜨리기 위해 직접 위조된 인스턴스를 만들어낼 수도 있다. <br>
    > `readobject` 메서드를 바깥 클래스에 추가하면 이 공격을 가볍게 막아낼 수 있다.

4. 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 `readResolve` 메서드

    > 이 메서드는 역직렬화 시 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해 줌 <br>
    > 즉, 역직렬화된 인스턴스는 일반 인스턴스와 마찬가지로 동일한 생성자, 정적 팩토리, 메서드를 사용해 생성된다. <br>
    > 따라서 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또 다른 수단을 강구하지 않아도 된다.

> 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다. <br>
> 앞서의 두 접근법과 달리, 직렬화 프록시는 `Period`의 필드를 `final`로 선언해도 되므로 `Period` 클래스를 진정한 불변으로 만들 수도 있다. <br>
> 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.

## EnumSet의 직렬화 프록시

```
private static class SerializationProxy <E extends Enum<E>> implements Serialzable {
	// EnumSet의 원소 타입
	private final Class<E> elementType;
	
	// EnumSet 안의 원소들
	private final Enum<?>[] elements;
	
	SerializationProxy(EnumSet<E> set) {
		elementType = set.elementType;
		elements = set.toArray(new Enum<?>[0]);
	}
	
	private Object readResolve() {
		EnumSet<E> result = EnumSet.noneOf(elementType);
		for(Enum<?> e : elements) 
			result.add((E) e);
		return result;
	}
	
	private static final long serialVersionUID = 83912381232;
}
```

## 한계

> 1. 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다. <br>
> 2. 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. <br>
> - 이런 객체의 메서드를 직렬화 프록시의 `readResolve` 안에서 호출하려 하면 `ClassCastException`이 발생할 것이다.
>  직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어진 것이 아니기 때문이다.
> 3. 직렬화 프록시 패턴이 주는 강력함과 안전성에도 대가는 따른다. 
> - Period 예를 내 컴퓨터 에서 실행해보니 방어적 복사 때보다 14%가 느렸다.