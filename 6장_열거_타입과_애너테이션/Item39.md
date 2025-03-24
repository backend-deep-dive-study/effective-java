# 명명 패턴보다 애너테이션을 사용하라

## 명명 패턴

- JUnit은 버전 3까지 테스트 메서드 이름을 test로 시작하게끔 했다.

### 단점

- 오타가 나서는 안된다.
    - JUnit 3에서는 test라고 적지 않으면 무시하고 지나치기 때문에 개발자는 통과했다고 오해한다.

- 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
    - 클래스이름이 Test이더라도 JUnit에서는 관심이 없고, 개발자가 의도한 테스트는 수행되지 않는다.

- 프로그램 요소를 매개변수로 전달한 마땅한 방법이 없다.
    - 특정 예외를 던져야만 성공하는 테스트의 기대 예외 타입을 매개변수로 전달할 수 없다.

## 애너테이션

- 애너테이션은 명명 패턴의 모든 문제를 해결해주는 개념으로, JUnit도 버전 4부터 전면 도입했다.

### 예제 - 기본 사용법

- 간단한 테스트용 애너테이션으로, 예외가 발생하면 해당 테스트를 실패로 처리하는 예제로 설명을 한다.

```java
import java.lang.annotation.*;

/**
 * 테스트 애너테이션 선언하는 애너테이션이다.
 * 메게변수 없는 정적 메서드 한정이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- **메타애너테이션(meta - annotation)**
    - 애너테이션 선언에 다는 애너테이션

    - @Retention(RetentionPolicy.RUNTIME)
        - @Test가 런타임에도 유지되어야 한다는 표시
        - 이 메타애너테이션을 생략하면 테스트 도구는 @Test를 인식할 수 없다.
    
    - @Target(ElementType.METHOD)
        - @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다.
        - 클래스 선언, 필드 선언 등 다른 프로그램 요소에는 달 수 없다.

- **마커(marker) 애너테이션**
    - @Test처럼 아무 매개변수 없이 단순히 대상을 마킹하기 위한 애너테이션을 의미한다.

- @Test 어노테이션 사용 예
    - 총 4개의 테스트 메서드 중 1개는 성공, 2개는 실패, 1개는 잘못 사용<br>
    @Test가 없는 나머지 4개의 메서드는 테스트 도구가 무시

    ```java
    public class Sample {
        @Test public static void m1() { } // 성공해야 한다.
        public static void m2() { }
        @Test public static void m3() { // 실패해야 한다.
            throw new RuntimeException("실패");
        }
        public static void m4() { }
        @Test public void m5() { } // 잘못 사용한 예: 정적 메서드가 아니다.
        public static void m6() { }
        @Test public static void m7() { // 실패해야 한다.
            throw new RuntimeException("실패");
        }
        public static void m8() { }
    }
    ```

    - @Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다.

- @Test를 처리하는 프로그램 - 테스트 도구

    ```java
    import java.lang.reflect.*;

    public class RunTests {
        public static void main(String[] args) throws Exception {
            int tests = 0;
            int passed = 0;
            Class<?> testClass = Class.forName(args[0]);
            for (Method m : testClass.getDeclaredMethods()) {
                if (m.isAnnotationPresent(Test.class)) { // @Test 애너테이션이 달린 메서드를 찾아주는 로직
                    tests++;
                    try {
                        m.invoke(null);
                        passed++;
                    } catch (InvocationTargetException wrappedExc) {
                        Throwable exc = wrappedExc.getCause();
                        System.out.println(m + " 실패: " + exc);
                    } catch (Exception exc) {
                        System.out.println("잘못 사용한 @Test: " + m);
                    }
                }
            }
            System.out.printf("성공: %d, 실패: %d%n",
                            passed, tests - passed);
        }
    }
    ```

    ```
    public static void Sample.m3() failed: RuntimeException: Boom
    Invalid @Test: public void Sample.m5()
    public static void Sample.m7() failed: RuntimeException: Crash 성공: 1, 실패: 3
    ```

### 예제 - 특정 예외를 던져야 성공하는 테스트를 지원하는 방법

- 애너테이션 타입
    ```java
    import java.lang.annotation.*;

    /**
    * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
    */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        Class<? extends Throwable> value();
    }
    ```

- 애너테이션 사용
    ```java
    public class Sample2 {
        @ExceptionTest(ArithmeticException.class)
        public static void m1() {  // 성공해야 한다.
            int i = 0;
            i = i / i;
        }
        
        @ExceptionTest(ArithmeticException.class)
        public static void m2() {  // 실패해야 한다. (다른 예외 발생)
            int[] a = new int[0];
            int i = a[1];
        }
        
        @ExceptionTest(ArithmeticException.class)
        public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
    }
    ```

- 애너테이션을 다루는 테스트 도구
    ```java
    if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
            m.invoke(null);
            System.out.println("테스트 %s 실패: 예외를 던지지 않음\n", m);
        } catch (InvocationTargetException wrappedExc) {
            Throwable exc = wrappedExc.getCause();
            // 수정 시작
            Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
            if (excType.isInstance(exc)) {
                passed++;
            // 수정 종료
            } else {
                System.out.println(
                    "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s\n",
                    m, excType.getName(), exc);
            }
        } catch (Exception exc) {
            System.out.println("잘못 사용한 @ExceptionTest: " + m);
        }
    }
    ```

### 예제 - 여러 개의 예외를 명시하고 하나가 발생하면 성공하는 테스트를 지원하는 방법

#### 방법 1 - 단순 배열로 변경

- 애너테이션 타입
    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        Class<? extends Throwable>[] value();
    }
    ```

- 애너테이션 사용
    ```java
    @ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
    public static void doublyBad() { // 성공해야 한다.
        List<String> list = new ArrayList<>();

        //자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
    ```

- 애너테이션을 다루는 테스트 도구
    ```java
    if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
            m.invoke(null);
            System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (Throwable wrappedExc) {
            Throwable exc = wrappedExc.getCause();
            // 수정 시작
            int oldPassed = passed;
            Class<? extends Throwable>[] excTypes = 
                m.getAnnotation(ExceptionTest.class).value();
            for (Class<? extends Throwable> excType : excTypes) {
                if (excType.isInstance(exc)) {
                    passed++;
                    break;
                }
            }
            if (passed == oldPassed)
                System.out.printf("테스트 %s 실패: %s %n", m, exc);
            // 수정 종료
        }
    }
    ```

#### 방법 2 - @Repeatable 메타애너테이션 사용 (자바 8 ~)

- @Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.

- @Repeatable 구현 주의 사항
    1. @Repeatable을 단 애너테이션을 반환하는 `컨테이너 애너테이션`을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전댈해야 한다.

    2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.

    3. 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.

- 반복 가능한 애너테이션 타입
    ```java
    // 반복 가능한 애너테이션
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Repeatable(ExceptionTestContainer.class)
    public @interface ExceptionTest {
        Class<? extends Throwable> value();
    }

    // 컨테이너 애너테이션
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTestContainer {
        ExceptionTest[] value();
    }
    ```

- 애너테이션 사용
    ```java
    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() { // 성공해야 한다.
        List<String> list = new ArrayList<>();

        //자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
    ```

- @Repeatable 처리 주의 사항

    - 반복 가능 애너테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 `컨테이너` 애너테이션 타입이 적용된다.
        - `getAnnotationByType` 메서드는 이 둘을 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져온다.
        - `isAnnotationPresent` 메서드는 이 둘을 명확히 구분한다.
    - 반복 가능 애너테이션을 여러 번 단 다음 `isAnnotationPresent`로 반복 가능 애너테이션이 달렸는지 검사하면 "그렇지 않다"라고 알려준다.(컨테이너가 달렸기 때문이다.)
    - `isAnnotationPresent`로 컨테이너 애너테이션이 달렸는지 검사한다면 반복 가능 애너테이션을 한 번만 단 메서드를 무시하고 지나친다.

    → 달려 있는 수와 상관없이 모두 검사하려면 둘을 따로따로 확인해야 한다.

- 애너테이션을 다루는 테스트 도구
    ```java
    // 수정 시작
    if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
    // 수정 종료
        tests++;
    }

    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        // 수정 시작
        ExceptionTest[] excTests = 
            m.getAnnotationsByType(ExceptionTest.class);
        for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
        // 수정 종료
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
    ```

- 반복 가능 애너테이션으로 가독성을 개선할 수 있다면 이 방식을 사용하자.
- 단, 애너테이션을 선언하고 처리하는 부분에서의 코드 양이 늘어나며, 특히 처리 코드가 복잡해져 오류가 날 가능성이 커질 수 있다.

## 결론

- 테스트는 애너테이션이 할 수 있는 극히 일부이다.
- 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일이라면 적당한 애너테이션 타입도 함께 정의해 제공하자.
- **애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.**
- 일반 프로그래머는 애너테이션 타입을 직접 정의할 일은 없지만, 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들을 사용해야 한다.

## 부록

### @Retention 종류
- RetentionPolicy.SOURCE: 소스 코드에만 존재하며, 컴파일러에 의해 무시된다.
- RetentionPolicy.CLASS: 클래스 파일에 포함되지만, 런타임에는 JVM에 의해 유지되지 않는다.(기본값)
- RetentionPolicy.RUNTIME: 클래스 파일에 포함되고 런타임에 JVM에 의해 유지된다. 리플렉션(Reflection)을 통해 접근 가능하다.

### @Target 종류
- ElementType.TYPE: 클래스, 인터페이스, 열거형에 적용
- ElementType.FIELD: 필드에 적용
- ElementType.METHOD: 메서드에 적용
- ElementType.PARAMETER: 메서드 매개변수에 적용
- ElementType.CONSTRUCTOR: 생성자에 적용
- ElementType.LOCAL_VARIABLE: 지역 변수에 적용
- ElementType.ANNOTATION_TYPE: 다른 어노테이션에 적용
- ElementType.PACKAGE: 패키지에 적용
- ElementType.TYPE_PARAMETER (Java 8+): 타입 매개변수에 적용
- ElementType.TYPE_USE (Java 8+): 타입 사용에 적용
- ElementType.MODULE (Java 9+): 모듈에 적용
- ElementType.RECORD_COMPONENT (Java 14+): 레코드 컴포넌트에 적용

### 애너테이션의 쓰임새
- 컴파일러를 위한 힌트 제공: 컴파일러가 특정 동작을 수행하거나 경고를 발생
    - @Override

- 코드 자동 생성: 소스 코드에서 자동으로 새로운 코드를 생성
    - @Getter, @Setter

- 런타임 동작 변경: 애너테이션이 붙은 클래스나 메서드를 런타임에 동적으로 처리
    - @Controller, @RequestMapping

- 보안 및 접근 제어: 보안 정책을 적용하거나 권한을 제어
    - @RolesAllowed("ADMIN")

- JSON/XML 직렬화 및 역직렬화
    - @JsonProperty("user_id")

- 데이터베이스 매핑(ORM)
    - @Entity, @Column

- 유효성 검사
    - @Valid, @NotNull, @Size