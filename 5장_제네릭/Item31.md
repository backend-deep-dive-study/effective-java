# 한정적 와일드카드를 사용해 API 유연성을 높이라

### 제네릭은 불공변

- `List<String>`은 `List<Object>`의 하위타입이 아님
- 불공변 방식보다 유연한 것의 필요성
    
    ```java
    public void pushAll(Iterable<E> src> {
    	for(E e : src)
    		push(e);
    }
    ```
    
    - Iterable src의 원소 타입이 스택의 원소 타입과 일치할 경우 잘 작동
    - Stack<Number> 선언 후 pushAll(integerValue) 호출
        - 제네릭은 불공변이라 에러 발생 → Integer가 Number의 하위X

### 한정적 와일드카드 타입

- 하위 or 상위 타입을 사용하도록 지원
- extends, super 사용
- pushAll : E의 하위 타입의 Iterable
    - 생산자 매개변수 : 하위 타입을 자유롭게 사용

```java
public void pushAll(Iterable<? extends E> src) { ... }
```

- popAll : E의 상위 타입의 Collection
    - 소비자 : 컬렉션에서 객체를 꺼내올 때 하위 타입까지 포괄할 수 있는 상위 타입으로 받아야함

```java
public void popAll(Collection<? super E> dst> { ... }
```

- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 사용

## 어떤 와일드카드 타입을 써야할까?

### PECS : producer-extends, consumer-super

- 생산자 매개변수 : `<? extends T>`  사용
    - 모두 E의 생산자 → PECS 공식에 따라 변경 필요
    
    ```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2)
    ```
    
    - 반환타입에는 한정적 와일드카드 사용 불가 → 클라이언트 코드에서도 와일드카드 사용 필요
    
    ```java
    public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2);
    ```
    
- 소비자 매개변수 : `<? super T>` 사용
- 클래스 사용자가 와일드카드 타입을 신경 써야 하면 API는 문제가 있을 가능성이 큼

### max 메서드 예시

- 기존
    
    ```java
    	public static <E extends Comparable<E>> E max(List<E> list) { ... }
    ```
    
    - `List<ScheduledFuture<?>> scheduledFutures` 처리 불가
        - ScheduledFuture가 `Comparable<ScheduledFuture>` 를 구현하지 않아서
        - ScheduledFuture는 Delayed의 하위 인터페이스 → Delayed는 `Comparable<Delayed>` 를 확장했음
        - ScheduledFuture의 인스턴스는 다른 ScheduledFuture 인스턴스와 Delayed 인스턴스와도 비교 가능하여 수정 전 max로 처리 불가
- PECS 공식 2번 적용 후
    
    ```java
    public static <E extends Comparable<? super E>> E max(List<? extends E> list) { ... }
    ```
    
    - 입력 매개변수 : E 인스턴스를 생산하므로 `List<? extends E>`
    - 타입 매개변수
        - Comparable<E> : E 인스턴스를 소비하므로 `Comparable<? super E>` (Comparable은 언제나 소비자)

### 제네릭과 와일드카드의 공통 부분

- 타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드 정의 시 어떤 것을 사용해도 괜찮은 때가 있음
- swap의 예시
    - 타입 매개변수 사용
        
        ```java
        public static <E> void swap(List<E> list, int i, int j)
        ```
        
    - 와일드카드 사용
        
        ```java
        public static void swap(List<?> list, int i, int j)
        ```
        
- 기본 규칙 : 메서드 선언에 타입 매개변수가 1번만 나오면 와일드카드로 대체
    - 비한정적 타입 매개변수 → 비한정적 와일드카드
    - 한정적 타입 매개변수 → 한정적 와일드카드

- 와일드카드 사용 swap 선언 시 문제
    
    ```java
    public static void swap(List<?> list, int i, int j){
    	list.set(i, list.set(j, list.get(i)));
    }
    ```
    
    - 꺼낸 원소를 리스트에 다시 넣을 수 없다는 오류 발생
    - 와일드카드의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 작성하여 활용
        
        ```java
        public static void swap(List<?> list, int i, int j){
        	swapHelper(list, i, j);
        }
        
        private static <E> void swapHelper(List<E> list, int i, int j){
        	list.set(i, list.set(j, list.get(i)));
        }
        ```
        
        - 리스트가 `List<E>` 임을 알고 있음
        - 꺼낸 값의 타입이 항상 E, E 타입의 값이라면 리스트에 넣어도 안전함을 알고 있음
    - 메서드 내부에서는 복잡한 제네릭 메서드를 사용하지만, swap을 호출해서 사용하는 클라이언트는 쉽게 사용 가능

### 정리

- 와일드카드 타입 적용 시 api가 유연해짐
- PECS 공식을 기억하자
    - 생산자 : extends
    - 소비자 : super
    - Comparable, Comparator는 모두 소비자!
