# Comparable을 구현할지 고려하라

## compareTo와 equeals의 차이

- compareTo는 Object의 equals와 2가지 성격만 빼면 동일하다.

   1. compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있다.

   2. compareTo는 제네릭하다.
      ```java
      public interface Comparable<T> {
         int compareTo(T o);
      }
      ```

      - 컴파일 타임 타입 안정성

      - 타입 캐스팅 불필요

      - 명확한 타입 관계

## compareTo 메서드의 일반 규약

- 객체가 주어진 객체보다
   - 작으면 음의 정수
   - 같으면 0
   - 크면 양의 정수
   - 비교할 수 없는 타임의 객체가 주어지면 `ClassCastException`

- 아래 규약은 모두 Comparable을 구현한 클래스를 가정한다.

1. 모든 x, y에 대해 sgn(x.comapreTo(y)) == -sgn(y.comapreTo(x))

   - x.compareTo(y)는 y.comapreTo(x)가 예외를 전질 때에 한해 예외를 던져야 한다.

2. 추이성을 보장해야 한다.

   - x.compareTo(y) > 0 && y.comapreTo(z) > 0이면 x.comapreTo(z) > 0이다.

3. x.comapreTo(y) == 0이면 sgn(x.comapreTo(z)) == sgn(y.comapreTo(z))다.

4. (x.compareTo(y) == 0) == (x.equals(y))

   - 필수는 아니지만 꼭 지키면 좋다.

   - 지켜지지 않으면 해당 클래스의 순서는 equals 메서드와 일관되지 않는다.

### 1번 규약 - 모든 x, y에 대해 sgn(x.comapreTo(y)) == -sgn(y.comapreTo(x))

- 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 얘끼이다.

   - 첫 번째 객체가 두 번째 객체보다 작으면, 두 번째가 첫 번째보다 커야 한다.

   - 첫 번째가 두 번째와 크기가 같다면, 두 번째는 첫 번째와 같아야 한다.

   - 첫 번째가 두 번째보다 크면, 두 번째는 첫 번째보다 작아야 한다.

### 2번 규약 - x.compareTo(y) > 0 && y.comapreTo(z) > 0이면 x.comapreTo(z) > 0

- 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다.

### 3번 규약 - x.comapreTo(y) == 0이면 sgn(x.comapreTo(z)) == sgn(y.comapreTo(z))

- 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.

### 4번 규약 - (x.compareTo(y) == 0) == (x.equals(y))

- 필수는 아니지만 꼭 지키길 권한다.

- comapreTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것이다.

- 지키지 않는다면 컬렉션이 구현한 인터페이스(Collection, Set, Map)에 정의된 동작과 엇박자

   - 동치성을 비교할 때 equals 대신 comapreTo를 사용하기 때문

## compareTo 구현 방법

- 관계 연산자 `<`와 `>`는 추천하지 않는다.

   - 자바 7부터 기본 타입 클래스들에 새로 추가된 정적 메서드 compare를 사용하자.

```java
@Override
public int comapreTo(Person p) {
   int result = Integer.compare(age, p.age);

   if (result == 0) {
      result = String.compare(name, p.name);

      if (result = 0)
         result = Integer.comapre(areaCode, p.areaCode);
   }
}
```

## Comparator 인터페이스

### 장단점

- 자바 8부터 Comparator를 사용하여 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.

- 코드의 간결함이 있지만 성능의 저하가 뒤따르니 조심해야 한다.

```java
private static final Comparator<Person> COMPARATOR = 
   comparingInt((Person p) -> p.age)
      .thenComparingString(p -> p.name)
      .thenComparingInt(p -> p.areaCode);

@Override
public int comapreTo(Person p) {
   return COMPARATOR.compare(this, p);
}
```

### 주의점

- 단순 `값의 차`를 기준으로 compareTo나 compare 메서드를 방식을 사용해서는 안된다.
   ```java
   static Comparator<Object> hashCodeOrder = new Comparator<>() {
      public int compare(Object o1, Object o2) {
         return o1.hashCode() - o2.hashCode();
      }
   };
   ```
   
   - 정수 오버플로를 일으키거나 부동소수점 계산 방식에 따른 오류가 날 수 있다.


- 반든시 아래 두 방식 중 하나를 사용해야 한다.

   ```java
   static Comparator<Object> hashCodeOrder = new Comparator<>() {
      public int compare(Object o1, Object o2) {
         return Integer.comapre(o1.hashCode(), o2.hashCode());
      }
   };
   ```

   ```java
   static Comparator<Object> hashCodeOrder = Compartor.comparingInt(o -> o.hashCode());
   ```
