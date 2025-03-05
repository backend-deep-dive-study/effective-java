# clone 재정의는 주의해서 진행하라

### Cloneable의 문제점
- java.lang.Cloneable : Java에서 객체를 복사할 수 있도록 해주는 마커 인터페이스
- Cloneable 자체는 아무 메서드도 가지지 않아 마커 인터페이스임
- clone() 메서드는 Object에서 상속되고 protected라서 외부 객체에서 clone 메서드를 호출할 수 없음
- 그러나 Cloneable을 구현하지 않으면 clone() 호출 시 CloneNotSupportedException 발생

- clone() 메서드
  - 객체의 복사본을 생성해 반환한다.
  - 필수 조건- > 새로운 인스턴스를 만들어 반환하고, 같은 클래스 타입을 유지해야 함.
    ```
    x.clone() != x
    x.clone().getClasss() == x.getClass()
    ```
  - 강제성이 없기 때문에 개발자가 직접 보장할 필요가 있음
  
> 🔍 **clone() 메서드**
> <br>기본적으로 얕은 복사 를 수행함
> <br>기본 타입인 경우는 값이 복사되지만 참조 타입인 경우 객체를 참조하게 됨
> <br>깊은 복사를 원한다면(가변 객체가 있는 경우)( clone()을 직접 구현

### Cloneable 구현
  - super.clone 호출 -> 원본 필드와 같은 값을 가짐
  - 불변 클래스는 clone 메서드를 제공하지 않는 게 좋음
  - 공변 반환 타입 (Convariant Return Type) : 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있음
  - 만약 clone 메서드가 가변 객체를 참조하는 순간 문제가 생길 수 있음
  - clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 함

  1. clone 재귀적 호출
    - 만약 final 필드로 선언한 경우 문제가 생길 수 있음
    - clone()은 필드를 메모리 단위로 복사함
    - 깊은 복사가 필요하면 final 필드를 사용하지 않는 것이 좋
  3. 반복자 사용
    - 재귀의 경우 스택오버플로 위험이 있음
    - HashTable.Entry의 deep copy 메서드를 사용하되, 재귀 대신 반복자를 써서 순회
  4. 원본 객체 상태를 다시 생성하는 고수준 메서드 호출
    - HashTable에서 새로운 배열로 초기화하고 다시 모든 쌍에 대해 put 메서드를 호출해 같은 배열로 만들어주기
    - 저수준에서 바로 처리할 때보다 느리고 필드 단위 객체 복사를 우회함

  - public인 clone 메서드는 throws절을 없애야 함
  - 상속용 클래스는 Cloneable을 구현해서는 안됨
    - 하위 클래스에서 구현 여부를 선택할 수 있게 하기
    - clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 하기
      ```java
      @Override
      protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
      }
      ```

  - Cloneable을 구현한 스레드 안전 클래스 작성 시 clone 메서드도 동기화해야 함


## 정리
- cloneable을 구현하는 모든 클래스는 clone을 재정의해야 함
- 접근제한자는 public으로, 반환 타입은 클래스 자신으로.
- super.clone 호출 후 모든 가변객체를 복사하고 복제본이 가진 객체 참조가 모두 복사된 객체를 가리키도록 해야 함
- Cloneable을 아직 구현하지 않았다면 복사 생성자나 복사 팩터리가 더 나음
  - 인터페이스 타입 인스턴스를 인수로 받을 수 있음 (변환 생성자, 변환 팩터리)
  ```java
    class Person {
      String name;
      Address address;
  
      public Person(String name, Address address) {
          this.name = name;
          this.address = address;
      }
  
      // 복사 생성자
      public Person(Person other) {
          this.name = other.name;
          this.address = new Address(other.address.city);
      }
  }
  
  Person p1 = new Person("Alice", new Address("Seoul"));
  Person p2 = new Person(p1);  // 복사 생성자를 이용한 객체 복사
  ```

## 부록
1. 기본 Cloneable - 얕은 복사
```java
class Address {
    String city;

    public Address(String city) {
        this.city = city;
    }
}

class Person implements Cloneable {
    String name;
    Address address; // 참조 타입 필드

    public Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone(); // 얕은 복사 수행
    }
}

public class CloneExample {
    public static void main(String[] args) throws CloneNotSupportedException {
        Person p1 = new Person("Alice", new Address("Seoul"));
        Person p2 = (Person) p1.clone();  // 복제 수행

        System.out.println("원본: " + p1.name + ", " + p1.address.city); 
        System.out.println("복사본: " + p2.name + ", " + p2.address.city);

        // 복제본의 주소 변경
        p2.address.city = "Busan";

        // 원본도 영향을 받음 (얕은 복사 문제)
        System.out.println("수정 후 원본: " + p1.address.city);  // Busan ❌
        System.out.println("수정 후 복사본: " + p2.address.city); // Busan
    }
}
```

2. 깊은 복사 -> clone 오버라이드, 참조 필드 복사
```java
class Person implements Cloneable {
    String name;
    Address address;

    public Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone(); // 얕은 복사 수행
        cloned.address = new Address(this.address.city); // 참조 객체도 복사 (깊은 복사)
        return cloned;
    }
}

public class CloneExample {
    public static void main(String[] args) throws CloneNotSupportedException {
        Person p1 = new Person("Alice", new Address("Seoul"));
        Person p2 = (Person) p1.clone();  // 깊은 복사 수행

        System.out.println("원본: " + p1.name + ", " + p1.address.city); 
        System.out.println("복사본: " + p2.name + ", " + p2.address.city);

        // 복제본의 주소 변경
        p2.address.city = "Busan";

        // 원본은 영향을 받지 않음 (깊은 복사 성공)
        System.out.println("수정 후 원본: " + p1.address.city);  // Seoul ✅
        System.out.println("수정 후 복사본: " + p2.address.city); // Busan
    }
}
```

![image](https://github.com/user-attachments/assets/37526ba4-d9b8-4317-8ea6-f6aa935db7c4)
