# 다른 타입이 적절하다면 문자열 사용을 피하라
## 문자열이 잘못 사용되는 경우
1. 파일, 네트워크, 키보드 입력으로부터 데이터를 받을 때
   - 받은 데이터가 수치형이라면 int, float, BigInteger 등 적당한 수치 타입으로 변환
   - '예/아니오' 질문의 답이라면 적절한 열거 타입 또는 boolean으로 변환 
2. 열거 타입 대체
   - 상수를 열거할 때는 문자열보다는 열거 타입이 낫다.
3. 혼합 타입 대체
   - 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 좋지 않음.
   ```java
   String compoundKey = className + "#" + i.next();
   ```
   - #이 두 요소 중 하나에서 쓰였다면 혼란스러운 결과 초래
   - 문자열을 파싱해야 해서 느리고, 오류 가능성 존재
   - String이 제공하는 기능에만 의존해야 함.
   - 전용 클래스를 새로 만들 것!
4. 권한을 표현
    ```java
    public class ThreadLocal {
        private ThreadLocal() { } // 객체 생성 불가
        
        // 현 스레드의 값을 키로 구분해 저장한다.
        public static void set(String key, Object value);
        
        // (키가 가리키는) 현 스레드의 값을 반환한다.
        public static Object get(String key);
    }
    ```
    - 문자열 키는 전역 이름공간에서 공유되므로 서로 다른 클라이언트가 동일한 키 문자열을 선택할 가능성 존재
    - 악의적인 클라이언트가 의도적으로 같은 키를 사용하여 다른 클라이언트 데이터 접근 가능
    ```java
    // 해결 방안
    public class ThreadLocal {
        private ThreadLocal() { } // 객체 생성 불가
        
        public static class Key { // 권한
            Key() { }
            
            // 위조 불가능한 고유 키를 생성한다.
            public static Key getKey() {
                return new Key();
            }
        }
        
        public static void set(Key key, Object value);
        public static Object get(Key key);
    }

    // 더 개선
    // 정적 메서드 제거 및 중첩 클래스 Key를 ThreadLocal로 변경
    // 매개변수화하여 타입안정성 확보
    public final class ThreadLocal<T> {
        public ThreadLocal() { } 
        public void set(T value);
        public T get();
    }
    ```
    - 위조 불가능
    - 타입 안전성

## 정리
- 적합한 데이터 타입이 있거나 새로 작성 가능한 경우, 문자열을 피하자.
- 문자열은 잘못 사용하면 번거롭고, 유연성이 떨어지며, 성능 저하, 오류 가능성이 크다.

## 부록
### 전역 이름 공간이란?
- 프로그래밍에서 식별자(변수명, 함수명, 클래스명 등)가 유일하게 구분되어야 하는 범위를 의미
#### 전역 이름 공간 특징
- 전체 프로그램에서 공유되는 영역
  - 모든 코드가 접근할 수 있고 영향을 줄 수 있는 공간.
  - 예시된 ThreadLocal 클래스의 경우, 문자열 키는 모든 애플리케이션 코드에서 공유
- 이름 충돌 가능성
  - 같은 이름을 사용하는 여러 코드가 있을 때 충돌이 발생
  - 서로 독립적으로 개발된 코드들이 같은 문자열 키("config", "data" 등)를 선택할 가능성이 높음
```java
// 모듈 A
ThreadLocal.set("userConfig", configA);

// 모듈 B (다른 개발자가 작성)
ThreadLocal.set("userConfig", configB); // 의도치 않게 A의 데이터를 덮어씀
```
