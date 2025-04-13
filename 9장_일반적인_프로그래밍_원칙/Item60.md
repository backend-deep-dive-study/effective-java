#  옵셔널 반환은 신중히 하라

### 메서드가 특정 조건에서 값을 반환할 수 없을 때

- Java 8 이전
    > 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 두 가지 있었다. 

    - 예외를 던진다

        > 예외는 진짜 예외적인 상황에서만 사용해야 하며(아이템 69) 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용도 만만치 않다.

    - (반환 타입이 객체 참조라면) null을 반환하는 것이다.

        > null을 반환하면 이런 문제가 생기지 않지만, 그 나름의 문제가 있다. null을 반환할 수 있는 메서드를 호출할 때는, 별도의 null 처리 코드를 추가해야 한다. <br>
        > null 처리를 무시하고 반환된 null 값을 어딘가에 저장해두면 언젠가 NullPointerException이 발생할 수 있다.

- Java 8 이후
    > `Optional<T>`는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 아무것도 담지 않은 옵셔널은 'empty'라고 말하고, 어떤 값을 담은 옵셔널은 'present' 라고 한다. 옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다.

    → 보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 `Optional<T>`를 반환하도록 선언하면 된다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다. <br>
    → 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다

### 예시
```
public static <E extends Comparable<E>> E max(Collection<E> c) {
	if(c.isEmpty())
		throw enw IllegalArgumentException("빈 컬렉션");
		
	E result = null;
	for(E e : c) {
		if(result == null || e.compareTo(result) > 0) {
			result = Objects.requireNonNULL(e);
		}
	}
	
	return result;
}
```
> 컬렉션이 비었으면 예외를 던진다.
> 
```
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	if(c.isEmpty())
		return Optional.empty();
		
	E result = null;
	for(E e : c) {
		if(result == null || e.compareTo(result) > 0) {
			result = Objects.requireNonNULL(e);
		}
	}
	
	return Optional.of(result);
}
```

- 옵셔널을 반환하도록 구현하기는 어렵지 않다.

    >  Optional.empty()로 만들고, 값이 든 옵셔널은 Optional.of (value) 로 생성했다.

**※ 주의 : 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자.** → 오류 발생 가능성 증가, 이중 체크 문제 (Optional이 null인지 체크하고 값을 체크해야하는 문제)

### 옵셔널 반환
> 옵셔널은 검사 예외와 취지가 비슷하다.(아이템 71) 즉, 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다. 반환 값이 없을 경우에 대한 처리를 사용자가 작성해야하므로 null 보다 안전하고 깔끔하게 처리할 수 있다. <br>
> 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환한다.

1. 기본 값 설정
```
int maxValue = max(list).orElse(0);  // 기본 값 설정
```
2. 원하는 예외 설정
```
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
3. 항상 값이 채워져있다고 가정하여 성능상의 이득을 볼 수 있다.
```
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

### 주의

- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다. (아이템 54)
- Optional은 새로운 객체를 생성하므로 박싱/언박싱에 따른 오버헤드가 있음 (성능이 중요한 메서드라면 기본 타입 특화 옵셔널(OptionalInt 등) 사용 고려)
    → 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.
- Optional을 반환하는 메서드에서 null을 반환하지 말 것
- 컬렉션의 원소 타입으로 사용하지 말 것 → 이중 체크 문제

### 결론
> 반환 값이 없을 수도 있는 메서드라면 옵셔널 사용을 고려해야한다. 하지만 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다.