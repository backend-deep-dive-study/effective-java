# 박싱된 기본 타입보다는 기본 타입을 사용하라

### 자바의 데이터 타입

- 기본 타입
- 참조 타입
- 박싱된 기본 타입: 기본 타입에 대응하는 참조 타입

### 기본 타입 vs 박싱된 기본 타입

| 기본 타입 | 박싱된 기본 타입 |
| --- | --- |
| 값만 가짐 | 식별성이라는 속성도 가짐 → 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있음 |
| 언제나 유효한 값 | 유효하지 않은 값 null을 가질 수 있음 |
| 시간, 메모리 효율적 | 상대적으로 비효율적 |

### 잘못 구현된 비교자

```java
Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i = j ? 0 ： 1);

naturalOrder.compare(new Integer(42), new Integer(42));
```

- 두 인스턴스의 값이 같아 0을 기대 → 실제로 1 출력
- (i < j) : 오토박싱된 Integer가 기본 타입으로 변환된 후 비교
- (i == j) : 객체 참조의 식별성 검사 → 값이 같더라도 다른 인스턴스라면 false(1) 반환
- 같은 객체를 비교하는 게 아니면 박싱된 기본 타입에 == 연산자 사용 시 오류 발생

### 올바른 비교자 구현

- 박싱된 Integer를 기본 타입 정수로 저장하여 모든 비교 수행

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
	int i = iBoxed, j = jBoxed; // 오토박싱
	return i < j ? -1 : (i = j ? 0 : l);
};
```

### 박싱된 타입의 잘못된 사용 - 1

기본 타입과 박싱된 타입을 혼용한 연산

```java
public class Unbelievable {
	static Integer i;
	public static void main(String[] args) {
		if (i = 42)
			System.out.printin("믿을 수 없군!");
	}
}
```

- i == 42 검사 시 NullPointerException 발생
- i가 int가 아닌 Integer이기 때문 → 초기값이 null
- 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀림
- Integer를 int로 바꿔주면 됨

### 박싱된 타입의 잘못된 사용 - 2

```java
public static void main(String[] args) {
	Long sum = 0L;
	for (long i = 0; i <= Integer,MAX_VALUE; i++) {
		sum += i;
	}
	System.out.printIn(sum);
}
```

- sum이 박싱된 타입이라 기본 타입인 i를 더할 때
    - 박싱과 언박싱을 반복
    - 계속 새로운 객체 생성중
- sum을 Long으로 선언하여 성능 저하

### 박싱된 기본 타입의 사용

- 컬렉션의 원소, 키, 값으로 사용
    - 컬렉션은 기본 타입 담을 수 X
- 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로 사용
    - 자바가 타입 매개변수로 기본 타입 지원 X
- 리플렉션을 통해 메서드 호출 시 사용

### 정리

- 기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 기본 타입 사용
- 오토박싱이 박싱된 기본 타입 사용 시 번거로움을 줄여주지만, 위험까지 없애주지 않음
- 박싱된 기본 타입 == 비교 시 식별성 비교
- 기본 타입과 박싱된 기본 타입 혼용 시 언박싱 됨 → NullPointerException 발생 가능
- 기본 타입을 박싱하는 작업은 필요 없는 객체를 생성하는 부작용 발생 가능
