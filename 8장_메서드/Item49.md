# 매개변수가 유효한지 검사하라

## 반드시 문서화하라

- 메서드와 생정자의 입력 매개변수의 값의 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다.

- 메서드 몸체가 실행되기 전에 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다.

- 매개변수 검사를 하지 않았을 때의 문제점

  1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.

  2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 수 있다.

  3. 메서드는 문제없이 수행됏지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 낼 수 있다.

  -> 매개변수 검사에 실패하면 실패 원자성(failure atomicity)을 어기는 결과를 낳을 수 있다.

- public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.(@throws 자바독 태그 사용)

  - 보통은 `IllegalArgumentException`, `IndexOutOfBoundsException`, `NullPointerException` 중 하나가 될 것이다.

- 매개변수의 제약을 문서화했다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다.

## 예시

```java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 * 
 * @param m 계수(양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0)
        throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    ... // 계산 수행
}
```

- 위 메서드는 m이 null이면 NullPointerException을 던진다.

  - 이 설명은 어디에도 없지만 BigInteger 클래스 수준에서 기술되어 있다.

  - 클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔하다.

## null 검사

- 자바 7에 추가된 java.util.Objects.requireNonNull 메서드 덕분에 null 검사를 수동으로 하지 않아도 된다.

  - 원하는 예외 메시지도 지정할 수 있고, 입력을 그대로 반환하므로 값 사용 동시에 null 검사가 가능하다.

  ```java
  this.strategy = Objects.requireNonNull(strategy, "전략");
  ```

  - 반환값은 무시하고 순수한 null 검사 목적으로 사용해도 된다.

- 자바 9에서는 Objects에 범위 검사 기능도 더해졌다.

  - `checkFromIndexSize`, `checkFromIndex`, `checkIndex`이며, null 검사 메서드만큼 유연하지는 않다.

  - 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐다.

  - 닫힌 범위(closed range; 양 끝단 값을 포함하는)는 다루지 못한다.

## 단언문(assert)

- public이 아닌 메서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다.

  - 공개되지 않은 메서드라면 제작자인 우리가 메서드가 호출되는 상황을 통제할 수 있어, 유효한 값만이 메서드에 넘겨지리라는 것을 보증할 수 있고, 그렇게 해야 한다.

```java
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;
  ... // 계산 수행행
}
```

- 단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다는 것이다.

  ```java
  Deque<Integer> queue = new ArrayDeque<>();
  queue.add(1);

  while (!queue.isEmpty()) {
    int size = queue.size();

    while (size-- > 0) {
      assert !queue.isEmpty(); //이 코드가 없다면 poll() 시점에 아래 경고가 생긴다.
                               //Unboxing of 'queue. poll()' may produce 'NullPointerException'

      int cur = queue.poll();
    }
  }
  ```

- 일반적인 유효성 검사와 다른 점이 있다.

  1. 실패하면 AssertionError를 던진다.
  
  2. 런타임에 아무런 효과도, 아무런 성능 저하도 없다.<br>
  (java 실행 시 --ea 혹은 --enableassertions 플래그를 설정하면 런타임에 영향을 준다.)

## 나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라

- 메서드가 직접 사용하지 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써야 한다.

- 클라이언트가 사용하려 하는 시점에 에러가 발생하기 때문에 어디서 가져왔는지 추적하기 어려워진다.

- 생성자는 이 원칙의 특수한 사례이며, 생성자 매개변수의 유효성 검사는 꼭 필요하다.

## 메서드 몸체 실행 전에 매개변수 유효성 검사의 예외

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때

- 계산 과정에서 암묵적으로 검사가 수행될 때

  - Collections.sort(List)에서 리스트 안의 객체들은 모두 상호 비교될 수 있어야 한다.

  - 만약 상호 비교될 수 없는 타입의 객체가 들어 있다면 ClassCastException은 던지게 된다.

  - 암묵적 유효성 검사에 너무 의존했다가 실패 원자성을 해칠 수 있으니 주의하자


# 부록

### 실패 원자성(failure atomicity)

- 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다.

  - 불변 객체로 설계하기

  - 작업 수행 전에 매개변수 유효성 검사하기

  - 객체의 임시 복사본에서 작업한 뒤, 성공 시에만 원본을 수정하기
  
  - 복구 코드를 작성하여 작업 전 상태로 되돌리기

### checkIndex

```java
public static int checkIndex​(int index, int length)
```

- Checks if the index is within the bounds of the range from 0 (inclusive) to length (exclusive).

- The index is defined to be out-of-bounds if any of the following inequalities is true:
  - index < 0
  - index >= length
  - length < 0, which is implied from the former inequalities

- Parameters:
  - index - the index
  - length - the upper-bound (exclusive) of the range
  
- Returns:
  - index if it is within bounds of the range

- Throws:
  - IndexOutOfBoundsException - if the index is out-of-bounds

### checkFromToIndex
```java
public static int checkFromToIndex​(int fromIndex, int toIndex, int length)
```

- Checks if the sub-range from fromIndex (inclusive) to toIndex (exclusive) is within the bounds of range from 0 (inclusive) to length (exclusive).

- The sub-range is defined to be out-of-bounds if any of the following inequalities is true:
  - fromIndex < 0
  - fromIndex > toIndex
  - toIndex > length
  - length < 0, which is implied from the former inequalities

- Parameters:
  - fromIndex - the lower-bound (inclusive) of the sub-range
  - toIndex - the upper-bound (exclusive) of the sub-range
  - length - the upper-bound (exclusive) the range

- Returns:
  - fromIndex if the sub-range within bounds of the range

- Throws:
  - IndexOutOfBoundsException - if the sub-range is out-of-bounds

### checkFromIndexSize
```java
public static int checkFromIndexSize​(int fromIndex, int size, int length)
```

- Checks if the sub-range from fromIndex (inclusive) to fromIndex + size (exclusive) is within the bounds of range from 0 (inclusive) to length (exclusive).

- The sub-range is defined to be out-of-bounds if any of the following inequalities is true:
  - fromIndex < 0
  - size < 0
  - fromIndex + size > length, taking into account integer overflow
  - length < 0, which is implied from the former inequalities

- Parameters:
  - fromIndex - the lower-bound (inclusive) of the sub-interval
  - size - the size of the sub-range
  - length - the upper-bound (exclusive) of the range

- Returns:
  - fromIndex if the sub-range within bounds of the range

- Throws:
  - IndexOutOfBoundsException - if the sub-range is out-of-bounds