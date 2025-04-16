# 문자열 연결은 느리니 주의하라

### 문자열 연결 연산자(+)
- 문자열 n개를 잇는 시간은 n^2에 비례한다.
- 문자열은 불변이라 두 문자열을 연결할 경우 양쪽 내용을 모두 복사하므로 성능 저하 발생

### StringBuilder
- append 메서드를 사용하는 것이 빠름

## 부록

### String
- 불변 객체 (Immutable Object)
  - 한 번 생성된 문자열은 변경할 수 없음
  - 문자열을 수정하는 것처럼 보여도, 실제로는 새로운 객체가 생성

```java
String s = "hello";
s = s + " world"; // 새로운 String 객체가 생성됨
```

- 장점
  - 간단한 문자열 처리에 적합
  - 불변성으로 인해 스레드 안전성 확보
- 단점
  - 반복적인 문자열 조작 시 성능 저하 (매번 새로운 객체 생성)

### StringBuilder
- 가변 객체 (Mutable Object)
  - 내부 버퍼를 사용하여 문자열을 직접 수정할 수 있습니다.

```java
StringBuilder sb = new StringBuilder("hello");
sb.append(" world"); // 원본 객체가 수정됨
```

- 장점
  - 빠른 성능 (특히 반복적인 문자열 조작에 유리)
  - 메모리 효율적

- 단점
  - 스레드 안전하지 않음 (멀티스레드 환경에서는 주의 필요)

### StringBuffer
- 가변 객체이며, 스레드 안전 (Thread-Safe)
  - StringBuilder와 거의 동일하지만, 동기화 처리가 되어 있음

```java
StringBuffer sb = new StringBuffer("hello");
sb.append(" world"); // 스레드 안전하게 문자열 수정
```

- 장점
  - 멀티스레드 환경에서도 안정적으로 사용 가능

- 단점
  - 동기화로 인해 StringBuilder보다 성능이 떨어질 수 있음 (단일 스레드 환경에서는 비효율적)


