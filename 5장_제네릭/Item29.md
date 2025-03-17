# 이왕이면 제네릭 타입으로 만들라

- 제네릭 타입을 만드는 것을 배워두면 그만한 값어치는 충분히 한다.

## 기본 스택

- 아래 코드를 기반으로 제네릭 타입으로 변환하는 과정을 살펴볼 것이다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    // 나머지 코드...
}
```

## 제네릭 스택

- 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 **클래스 선언에 타입 매개 변수를 추가**하는 일이다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    // 나머지 코드...
}
```

### 오류 발생

- 이 단계에서 대체로 하나 이상의 오류나 경고가 나온다.

    - 이 클래스도 예외가 아니며, 하나의 오류가 발생했다.

    ```
    Stack.java:8: generic array creation
        elements = new E[DEFAULT_INITIAL_CAPACITY];
                   ^
    ```

- 아이템 28에서 설명했듯이, E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.

### 해결책

- 이 문제의 해결책은 두 가지이다.

1. 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법

    - Object 배열을 생성한 다음 제네릭 배열로 형변환

    - 이 방식으로 컴파일러는 오류 대신 경고를 내보낸다.

    ```
    Stack.java:8: warning: [unchecked] unchecked cast
    found: Object[], required: E[]
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
                       ^
    ```

    - 컴파일러는 타입 안전성을 증명할 수 없지만 우리는 가능하다.

    - 우리 스스로 타입 안전성을 해치지 않음을 확인하고 @SuppressWarnings 애너테이션으로 경고를 숨긴다.

    ```java
    // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서 타입 안전성을 보장하지만,
    // 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
    ```

2. elements 필드의 타입을 E[]에서 Object[]로 바꾸기

    - 이 방식은 첫 번째와 다른 오류가 발생한다.

    ```
    Stack.java:19: incompatible types
    found: Object, required: E
        E result = elements[--size];
                           ^
    ```

    - E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.

    - 이번에도 마찬가지로 우리가 직접 증명하고 경고를 숨길 수 있다.

    ```java
    // 비검사 경고를 적절히 숨긴다
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        E result = elements[--size];

        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
    ```

- 두 가지 방법 모두 나름의 지지를 얻고 있다.

    - 첫 번째 방법은 가독성이 더 좋다. 형변환을 배열 생성 시 단 한 번만 해주면 된다.<br>
    그에 비해, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다.

    - 첫 번째 방법은 (E가 Object가 아닌 한) 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킨다.<br>
    힙 오염이 맘에 걸리는 프로그래머는 두 번째 방식을 고수하기도 한다.

## 마무리 추가 설명

- Stack 예는 "배열보다는 리스트를 우선하라"는 아이템 28과 모순돼 보인다.
    
    - 하지만 자바가 리스트를 기본 타입으로 제공하지 않으므로 결국은 기본 타입인 배열을 사용해 구현해야 한다.

    - HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

- 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다.

    - Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack 등 모두 가능하다.

    - 단, Stack<int>나 Stack<double>과 같이 기본 타입은 사용할 수 없다.(컴파일 오류 발생)<br>
    (박싱된 기본 타입을 사용해 우회 가능)

- 타입 매개변수에 제약을 두는 **한정적 타입 매개변수(bounded type parameter)**

    ```java
    // concurrent.DelayQueue의 일부
    class DelayQueue<E extends Delayed> implements BlockingQueue<E>
    ```

    - <E extends Delayed>는 java.util.concurrent.Delayed의 하위 타입만 받는다는 뜻이다.<br>
    (물론 모든 타입은 자기 자신의 하위 타입이므로 DelayedQueue<Delayed>로도 사용 가능하다.)

    - 이 덕에 DelayQueue의 원소에서 (형변환 없이) 곧바로 Delayed 클래스의 메서드를 호출할 수 있다.

# 부록

## 힙 오염(heap pollution)

- 제니릭 타입의 변수가 실제로는 다른 타입의 객체를 참조하게 되는 상황을 말한다.

```java
List<String> stringList = new ArrayList<>();
List rawList = stringList;
rawList.add(42); // 컴파일러는 경고를 표시하지만 실행 가능함
String s = stringList.get(0); // ClassCastException 발생!
```

- 발생 원인

    1. 원시 타입(raw types)의 사용

    2. 비검사 경고(unchecked warnings)를 무시하는 경우

    3. 가변 인자(varargs)와 제네릭을 함께 사용하는 경우