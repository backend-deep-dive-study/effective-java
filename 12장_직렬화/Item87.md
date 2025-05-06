# 커스텀 직렬화 형태를 고려해보라
## 기본 직렬화 형태의 한계점
- 개발 일정이 촉박한 경우, 당장은 동작만 하는 API 설계에 집중하는게 일반적인 전략
- 단, 클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면,  
  향후 릴리스 때 구현을 변경할 수 없는 문제가 생김
## 기본 직렬화 형태가 적합한 케이스
- 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태도 무방
```java
public class Name implements Serializable {
    /**
    * 성. null이 아니어야 함.
    * @serial
    */
    private final String lastName;
    /**
    * 이름. null이 아니어야 함.
    * @serial
    */
    private final String firstName;
    /**
    * 중간이름. 중간이름이 없다면 null.
    * @serial
    */
    private final String middleName;
}
```
- 논리적 내용: 사람의 이름은 개념적으로 '이름', '성', '중간이름'으로 구성
- 물리적 표현: 클래스는 정확히 이 세 가지 문자열 필드로 구성
## 기본 직렬화 형태가 적합하지 않은 케이스
- 물리적 표현과 논리적 내용이 다른 경우
```java
public class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
}
```
- 논리적 내용: 이 클래스는 논리적으로 단순한 '문자열 목록'을 표현
- 물리적 표현: 물리적으로는 이중 연결 리스트를 사용하여 구현
## 물리적 표현과 논리적 표현 차이가 클 때 기본 직렬화 형태를 사용할 때 문제점
### 1. 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
- 내부 구현이 공개 API가 되어버려 구현을 변경해도 이전 방식을 계속 지원해야 함.
### 2. 너무 많은 공간을 차지할 수 있다.
- 내부 구현 세부사항까지 모두 저장되어 디스크 저장이나 네트워크 전송 효율이 떨어짐.
### 3. 시간이 너무 많이 걸릴 수 있다.
- 직렬화 로직이 객체 그래프를 직접 순회해야하기 때문에 오래 걸림.
### 4. 스택 오버플로를 일으킬 수 있다.
- 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이는 중간 정도 크기의 객체 그래프에서도 스택 오버플로가 발생할 수 있음.
## 커스텀 직렬화 형태 구현 방법
```java
public final class StringList implements Serializable {
    // 이전에는 size, head, 모든 Entry 객체, 연결 리스트 전체 구조가 저장됨.
    private transient int size = 0;
    private transient Entry head = null;
    
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) { ... }
    
    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        // 기본 직렬화를 수행하지만, transient가 아닌 필드가 없으므로 아무것도 저장하지 않음
        s.defaultWriteObject();
        // 리스트의 크기(문자열 개수)를 저장
        s.writeInt(size);
        // 모든 원소를 올바른 순서로 기록
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data); // 각 문자열 데이터만 저장

        // 따라서, 직렬화되는 부분은 리스트 크기, 문자열 데이터들의 순서 목록
    }
    
    // 역직렬화
    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt(); // 저장된 크기를 읽음
        // 문자열들을 읽어서 리스트에 추가
        for (int i = 0; i < numElements; i++)
            // add 메서드를 통해 내부 연결 리스트 구조를 재구성
            add((String) s.readObject()); 
    }
}
```
- defaultWriteObject와 defaultReadObject를 호출하는 이유
  - 향후 transient가 아닌 인스턴스 필드가 추가되더라도 상위와 하위 모두 호환되기 위함.
## 직렬화 시 유의점
- transient로 선언하지 않은 모든 인스턴스 필드가 직렬화됨.
  - 다른 필드에서 유도되는 필드나 JVM을 실행할 때마다 값이 달라지는 필드 등은 transient로 선언.
  - ex) 해시테이블 : 저장하는 위치를 키에서 구한 해시코드가 결정하는데 계산 방식은 구현에 따라 달라짐.
- transient 필드들은 역직렬화할 때 기본값으로 초기화됨.
  - 객체 참조 필드는 null, 숫자 기본 타입 필드는 0, boolean은 false
  - 기본값을 사용하면 안되는 경우 readObject 메서드에서 적절한 값으로 복원할 것.
- 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 함.
  - 모든 메서드를 synchronized로 선언한 스레드 안전 객체에서는 writeObject도 synchronized로 선언해야 함.
- 어떤 직렬화 형태든 직렬화 가능 클래스 모두 직렬 버전 UID를 명시적으로 부여할 것.
  - 잠재적인 호환성 문제가 사라짐.
  - 성능이 조금 빨라짐.
    - 직렬 버전 UID를 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산 수행
  ```java
    private static final long serialVersionUID = <무작위로 고른 long 값>;
  ``` 
  - 새로 작성하는 클래스에서는 아무 값이나 사용해도 되고 고유할 필요도 없음.
  - 기존 클래스의 경우 serialver 유틸리티를 통해 자동 생성된 값을 사용할 수 있음.
  - UID를 변경하면 역직렬화 시 InvalidClassException 발생 → 하위 호환을 깨뜨리고 싶을 때 사용.
  - 구버전으로 직렬화된 인스턴스들과의 호환성을 끊는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말 것
## 정리
> - 클래스를 직렬화 할 때 어떤 직렬화 형태를 사용할지 신중하게 결정할 것
> - 기본 직렬화 형태는 객체의 논리적 표현에 부합할 때만 사용
> - 그 외에는 객체를 적절히 설명하는 커스텀 직렬화 형태를 고려
> - 공개 메서드를 설계할 때처럼 시간을 들여 설계할 것 -> 향후 변경하기 어려움.
## 부록
### @serial
- 자바 직렬화 관련 문서화를 위한 자바독 태그.
- 직렬화될 수 있는 클래스의 private 필드에 대한 문서화를 위해 사용
- 해당 필드가 객체의 직렬화 형태에 포함된다는 것을 명시하고, 자바독이 이 정보를 "직렬화된 형태" 섹션에 포함시키도록 함.
### @serialData
- 자바독 유틸리티에게 이 내용을 직렬화 형태 페이지에 추가하도록 요청하는 역할
### transient
- transient로 표시된 필드는 객체를 직렬화할 때 JVM에 의해 무시.
- 즉, 해당 필드의 값은 직렬화에서 제외됨.
- 역직렬화 시 기본값으로 초기화됨.
