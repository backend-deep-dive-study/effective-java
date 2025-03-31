# 표준 함수형 인터페이스를 사용하라

- java.util.funciton 패키지에 필요한 용도에 맞는 **함수형 인터페이스**가 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.

  - API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다.

  - 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아진다.

- 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말라.

## 6가지 기본 인터페이스

- java.util.function 패키지에는 43개의 인터페이스가 담겨 있지만 기본 인터페이스 6개만 기억하면 나머지는 충분히 유추가 가능하다.

1. **Operator** 인터페이스 2개

  - 반환값과 인수의 타입이 같은 함수

    - 인수가 1개인 `UnaryOperator`와 2개인 `BinaryOperator`로 나뉜다.

2. **Predicate** 인터페이스

  - 인수 하나를 받아 boolean을 반환하는 함수

3. **Function** 인터페이스

  - 인수와 반환 타입이 다른 함수

4. **Supplier** 인터페이스

  - 인수를 받지 않고 값을 반환하는 함수(제공)

5. **Consumer** 인터페이스

  - 인수를 하나 받고 반환값은 없는 함수(소비)

| 인터페이스 | 함수 시그니처 | 예 |
|------------|--------------|------|
| UnaryOperator<T> | T apply(T t) | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add |
| Predicate<T> | boolean test(T t) | Collection::isEmpty |
| Function<T,R> | R apply(T t) | Arrays::asList |
| Supplier<T> | T get() | Instant::now |
| Consumer<T> | void accept(T t) | System.out::println |

## 변형

### 기본 타입을 반환하하는 변형 (27)

- 기본 인터페이스는 기본 타입인 int, long, doulbe용으로 각 3개씩 변형이 생긴다.

  - 예) int를 받는 Predicate → `IntPredicate`

- Function 인터페이스는 기본 타입을 반환하는 변형이 9개가 더 있다.

  - 인수와 반환 타입이 모두 기본 타입이라면 접두어로 `SrcToResult`를 사용한다. (총 6개)

    - 예) long을 받아 int를 반환하는 Function → `LongToIntFunction`

  - 입력이 객체 참조이고 결과가 int, long, double인 변형은 접두어로 `ToResult`를 사용한다. (총 3개)

    - 예) int[] 인수를 받아 long을 반환하는 Function → `ToLongFunction<int[]>`

### 인수 2개씩 받는 변형 (총 9개)

- `BiPredicate<T, U>`, `BiFunction<T, U, R>`, `BiConsumer<T, U>` 3가지가 있다.

- BiFunction에는 다시 기본 타입을 반환하는 세 변형이 존재한다.

  - `ToIntBiFunction<T, U>`

  - `ToLongBiFunction<T, U>`

  - `ToDoubleBiFunction<T, U>`

- Consumer에도 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형이 3개가 존재한다.

  - `ObjDoubleConsumer<T>`

  - `ObjIntConsumer<T>`

  - `ObjLongConsumer<T>`

### Supplier의 변형 (총 1개)

- boolean을 반환하도록 한 Supplier의 변형인 `BooleanSupplier`

## 예외 사항

- 대부분 상황에서는 함수형 인터페이스를 사용하는 것이 좋지만, 직접 작성해야 하는 경우도 있다.

- 용도에 맞는 게 없다면 직접 작성해야 한다.

  - 예) 매개변수 3개를 받는 경우, 검사 예외를 던지는 경우

- 구조적으로 동일한 경우에도 직접 작성해야 하는 경우가 있다.

  - 이 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는지 고민해야 한다.

  1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.

  2. 반드시 따라야 하는 규약이 있다.

  3. 유용한 디폴트 메서드를 제공할 수 있다.

## @FunctionalInterface

- 전용 함수형 인터페이스 작성하게 된다면 주의해서 작성해야 한다.

- 이를 예방하는 것이 @FunctionalInterface이며 3가지 목적이 있다.

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.

2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.

3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

- **직접 만든 함수형 인터페이스에는 항상 이 애너테이션을 사용하자**