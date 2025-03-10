# 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

## 상속용 클래스 설계 시 주의사항

- 상속용 클래스는 재정의할 수 있는 메서드들을 **내부적으로 어떻게 이용하는지(자기사용)** 문서로 남겨야 한다.

   > 재정의 가능 메서드란? public과 protected 메서드 중 final이 아닌 모든 메서드

   - 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다.

      - 재정의 가능 메서드를 호출하는 메서드의 API 설명에 적시해야 한다.

      - 추가로 어떤 순서로 호출하는지, 호출 결과가 이어지는 처리에 대한 영향도 담아야 한다.

- Implementation Requirements

   - 이 구절은 API 문서의 메서드 설명 끝에 존재하며 그 메서드의 내부 동작 방식을 설명하는 곳이다.
      
      - 메서드 주석에 @implSpec 태그를 붙여주면 자바독 도구가 생성해준다.

   - java.util.AbstractCollection

      ```
      public boolean remove（Object o）

      주어진 원소가 이 컬렉션 안에 있다면 그 인스턴스를 하나 제거한다（선택적 동작）.
      더 정확하게 말하면, 이 컬렉션 안에 'Object.equals(o, e)가 참인 원소' e가 하나 이상 있다면 그 중 하나를 제거한다.
      주어진 원소가 컬렉션 안에 있었다면(즉, 호출 결과 이 컬렉션이 변경됐다면) true를 반환한다.

         Implementation Requirements 이 메서드는 컬렉션을 순회하며 주어진 원소를 찾도록 구현되었다.
      주어진 원소를 찾으면 반복자의 remove 메서드를 사용해 컬렉션에서 제거한다.
      이 컬렉션이 주어진 객체를 갖고 있으나, 이 컬렉션의 iterator메서드가 반환한 반복자가 remove 메서드를 구현하지 않았다면 UnsupportedOperationException을 던지니 주의하자.
      ```

   - 위 예시를 통해 iterator 메서드를 재정의하면 remove 메서드의 동작에 영향을 줌을 알 수 있다.

## 하위 클래스 작성을 위한 지원

- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.
   - 드물게 protected 필드로 공개할 수도 있다.

- java.util.AbstractList

   ```
   protected void removeRange(int fromlndex, int tolndex)

   fromlndex(포함)부터 tolndex(미포함(까지의 모든 원소를 이 리스트에서 제거한다. 
   tolndex 이후의 원소들은 앞으로 (index만큼씩) 당겨진다.
   이 호출로 리스트는 'tolndex - fromlndex'만큼 짧아진다.
   (tolndex == fromlndex라면 아무런 효과가 없다.)
      이 리스트 혹은 이 리스트의 부분리스트에 정의된 clear 연산이 이 메서드를 호출한다.
   리스트 구현의 내부 구조를 활용하도록 이 메서드를 재정의하면 이 리스트와 부분리스트의 clear 연산 성능을 크게 개선할 수 있다.

      Implementation Requirements: 이 메서드는 fromlndex에서 시작하는 리스트 반복자를 얻어 모든 원소를 제거할 때까지 Listiterator.next와 Listiterator.remove를 반복 호출하도록 구현되었다.
   주의: Listiterator.remove가 선형 시간이 걸리면 이 구현의 성능은 제곱에 비례한다.
   
   Parameters:
      fromlndex   제거할 첫 원소의 인덱스
      tolndex     제거할 마지막 원소의 다음 인덱스
   ```

   - List 구현체의 최종 사용자는 removeRange 메서드에 관심이 없다.

   - 이 메서드가 제동된 이유는 하위 클래스에서 부분리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위함이다.<br>
   (removeRange가 없다면 하위 클래스에서 clear 메서드를 호출하면 제곱에 비례해 성능이 느려지거나 부분리스트의 메커니즘을 처음부터 새로 구현)

- 상속용 클래스에서 어떤 메서드를 protected로 노출해야 하는가?

   - 심사숙고해서 잘 예측해본 다음, 실제 하위 클래스를 만들어 시험해보는 것이 최선이다.

      - protected 수는 가능한 적어야 하지만 너무 적어서 상속으로 얻는 이점이 없어지지 않도록 주의해야 한다.

   - **상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다.**

- **상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.**

   - 최소 3개 이상 하위 클래스를 만들어야 하며, 하나 이상은 제 3자가 작성해야 한다.

## 상속 설계의 제약사항

- **상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.**

   ```java
   public class Super {
      // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
      public Super() {
         overrideMe();
      }

      public void overrideMe() {
      }
   }

   public final class Sub extends Super {
      // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
      private final Instant instant;

      Sub() {
         instant = Instant.now();
      }

      // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
      ©Override public void overrideMe() {
         System.out.printIn(instant);
      }

      public static void main(String[] args) {
         Sub sub = new Sub();
         sub.overrideMe();
      }
   }
   ```

   - 위 코드는 첫 번째는 null을 출력한다.
   
      - 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출하기 때문이다.

      - 만약 overrideMe에서 instant 객체의 메서드를 호출하려 했다면 `NullPointerException`이 던져졌을 것이다.


- Cloneable과 Serializable은 상속용 설계의 어려움을 추가한다.

   - clone과 readObject 메서드는 생성자와 비슷한 효과를 내기에 해당 두 메서드도 재정의 가능 메서드를 호출하면 안 된다.

   - Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 private이 아닌 protected로 선언해야 한다.

## 상속을 금지하는 방법

- 상속용으로 설계하지 않은 클래스는 상속을 금지하자.

- 상속을 금지하는 2가지 방법

   1. 클래스를 final로 선언하기

   2. 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리를 만들기

### 상속 금지에 대한 논란

- 인터페이스가 있다면 상속을 금지해도 문제가 없다.

   - 예: Set, List, Map

- 래퍼 클래스 패턴이 상속 대신 쓸 수 있는 더 나은 대안이다.

- 구체 클래스가 표준 인터페이스를 구현하지 않았는데 상속을 금지하면 사용하기에 상당히 불편해진다.

   - 이런 클래스에 상속을 허용하는 합당한 방법이 하나 있다.

      - 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서로 남기는 것<br>
      (재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽히 제거하기)

   - 클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거하는 기계적인 방법

      1. 각각의 재정의 가능 메서드는 자신의 본문 코드를 private '도우미 메서드'로 옮기기

      2. 각각의 재정의 가능 메서드에서 도우미 메서드를 호출하도록 수정하기

      3. 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 직접 호출하도록 수정하기