# 스레드 안전성 수준을 문서화하라
## 스레드 안전성 문서화의 중요성
- 스레드 안전성은 클래스와 클라이언트 사이의 중요한 계약
- API 문서에 스레드 안전성 수준이 명시되지 않으면 사용자가 잘못된 가정을 하여 오류를 유발할 수 있음.
- synchronized 한정자만으로는 스레드 안전성을 보장한다고 전달할 수 없음.
  - 자바독 기본 설정에는 synchronized가 문서에 나타나지 않음.
  - synchronized 여부는 구현 세부사항일 뿐, API에 속하지 않음.
## 스레드 안전성 수준
### 1. 불변(Immutable)
> - 상태가 변하지 않아 동기화가 필요 없음
> - ex) String, Long, BigInteger
### 2. 무조건적 스레드 안전(Unconditionally Thread-Safe)
> - 내부적으로 모든 동기화를 처리 -> 외부 동기화 불필요
> - ex) AtomicLong, ConcurrentHashMap
### 3. 조건부 스레드 안전(Conditionally Thread-Safe)
> - 대부분은 안전하지만, 일부 메서드는 외부 동기화 필요
> - ex) Collections.synchronizedXXX() 컬렉션들
### 4. 스레드 안전하지 않음(Not Thread-Safe)
> - 클라이언트가 직접 외부 동기화 필요
> - ex) ArrayList, HashMap
### 5. 스레드 적대적(Thread-Hostile)
> - 외부 동기화로도 안전하지 않음. 일반적으로로 정적 데이터를 아무 동기화 없이 수정.
> - ex) 잘못 구현된 클래스
## 문서화 시 유의사항
- **클래스**에 스레드 안전성 수준을 문서화 할 것
- 조건부 스레드 안전 클래스인 경우
  - 어떤 메서드 호출 순서에 외부 동기화가 필요한지.
  - 그 순서로 호출하기 위해 어떤 락을 필요로 하는지 문서화 할 것
- @Immutable, @ThreadSafe, @NotThreadSafe 같은 애너테이션을 활용하면 됨.
- **독특한 특성의 메서드**인 경우, 해당 메서드에 문서화 할 것
  ```java
    // synchronizedMap은 스레드 안전한 맵
    Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
    Set<K> s = m.keySet(); // 안전
    synchronized(m) {
        // for-each 루프는 내부적으로 Iterator를 이용하기 때문에 호출 중에 맵이 수정되면 ConcurrentModificationException이 발생
        // 즉, Iterator는 자체적으로 동기화를 보장하지 않음!
        for (K key : s) process(key);
    }
  ``` 
## 서비스 거부 공격을 막기 위한 비공개 락
- 서비스 거부 공격 : 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 것
- synchronized 메서드 대신 비공개 락 객체를 사용해야 함.
```java
// private이라 클래스 외부 접근 불가
// final 락 객체가 다른 객체로 교체될 수 없음
private final Object lock = new Object(); 

public void foo() {
    synchronized(lock) { // lock 객체를 기준으로 동기화되게 됨.
        // ...
    }
}
```
- 하위 클래스가 내부 동기화에 접근 불가능하므로 오작동을 방지.
- 락 객체는 final로 선언해 불변성을 보장
## 정리
> - 모든 클래스는 스레드 안전성 수준을 문서화해야함.
> - 조건부 스레드 안전 클래스는 어떤 메서드 순서에 외부 동기화가 필요하고 어떤 락을 얻어야 하는지 알려줘야 함.
> - 무조건적 스레드 안전 클래스를 작성할 때는 synchronized 메서드가 아닌 비공개 락 객체를 사용하여 클라이언트나 하위 클래스가 동기화 메커니즘을 깨뜨리는 것을 방지해야함.
