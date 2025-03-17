## 인터페이스는 타입을 정의하는 용도로만 사용하라
### 인터페이스의 올바른 용도
- 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할.
- 클래스가 인터페이스를 구현한다는 것은 인스턴스로 무엇을 할 수 있는지 클라이언트에게 알려주는 것.
### 올바르지 않은 인터페이스 활용
#### 상수 인터페이스 안티패턴
- 메서드 없이 static final 필드(상수)로만 가득 찬 인터페이스
- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아닌 내부 구현에 해당함!
- 내부 구현을 클래스의 API로 노출하는 행위로, 사용자에게 혼란을 주고 클라이언트 코드가 내부 구현에 종속되게 함.
- java.io.ObjectStreamConstants와 같은 자바 플랫폼 라이브러리에서도 상수 인터페이스가 있으나, 따라해서는 안 됨.
### 상수를 공개하는 좋은 방법
- 특정 클래스/인터페이스와 강하게 연관된 상수인 경우
  - 해당 클래스나 인터페이스 자체에 추가
  - ex) Integer의 MIN_VALUE 또는 MAX_VALUE
- 열거 타입으로 나타내기 적합한 상수
  - 열거 타입(enum)으로 만들어 공개
- 그 외의 경우
  - 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개
### 유틸리티 클래스 사용 팁
- 유틸리티 클래스의 상수를 빈번히 사용할 경우 정적 임포트(static import)로 클래스 이름 생략 가능
```java
import static effectivejava.chapter4.item22.constantutilityclass.PhysicalConstants.*;
 public class Test {
 double atoms(double mols) { 
return AVOGADROS_NIJMBER * mols;
 }
 // Physicalconstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다, 
}
```
### 핵심 정리
- 인터페이스는 타입을 정의하는 용도로만 사용할 것!
- 상수 공개용 수단으로 사용하지 말자.

### 느낀점
- 인터페이스는 메서드들의 구현을 강제하는 역할로만 알고 있었지만 상수 저장소나 기타 다른 목적으로 잘못 쓰이는 경우도 있다는 것을 알게 되었습니다. 

## 추가
### "특정 클래스/인터페이스와 강하게 연관된 상수"를 해당 클래스나 인터페이스에 추가하라는 의미
- 순수하게 상수만 담은 인터페이스를 만들지 말고 이미 존재하는, 분명한 타입 정의 역할을 하는 클래스나 인터페이스에 그 타입의 개념과 직접 연관된 상수만 추가하라는 의미!
```java
// 상수 인터페이스 안티패턴
public interface MathConstants {
    double PI = 3.14159265358979323846;
    double E = 2.7182818284590452354;
    double GOLDEN_RATIO = 1.6180339887498948482;
}
// 인터페이스가 특정 타입을 정의하는 본래 목적을 가짐
public interface Shape {
    double area();
    double perimeter();
    
    // 이 인터페이스와 직접 관련된 상수는 괜찮음
    double PI = 3.14159265358979323846;
}
```
### 안좋은 예시들 추가
#### 1. 구현을 강제하는 마커 인터페이스
```java
// 아무 메서드도 없는 마커 인터페이스
public interface Persistent {
    // 메서드 없음 - 단지 "이 클래스는 영속적이다"를 표시하기 위함
}

// 실제 구현이 필요 없는데도 인터페이스를 구현해야 함
public class User implements Persistent {
    private String username;
    private String email;
    // ...
}
```
#### 2. 구현 코드를 포함하는 인터페이스(Java8이전 방식)
```java
public interface DataProcessor {
    void process(Data data);
    
    // 실제 구현 코드를 포함하는 유틸리티 메서드
    default Data convert(String input) {
        // 구현 코드가 여기에 있음
        return new Data(input);
    }
}
```
#### 3. 너무 큰 인터페이스 (인터페이스 분리 원칙 위반)
```java
// 너무 많은 책임을 가진 인터페이스
public interface SuperService {
    void saveToDatabase(Object data);
    void sendEmail(String to, String subject, String body);
    void generateReport(ReportType type);
    void processPayment(Payment payment);
    void calculateTax(Order order);
    // ... 더 많은 메서드들
}

// 클래스는 필요하지 않은 메서드까지 모두 구현해야 함
public class TaxCalculator implements SuperService {
    // 실제로는 calculateTax만 필요한데 다른 메서드도 모두 구현해야 함
    @Override
    public void saveToDatabase(Object data) { /* 불필요한 구현 */ }
    
    @Override
    public void sendEmail(String to, String subject, String body) { /* 불필요한 구현 */ }
    
    // ... 다른 모든 메서드 구현
}
```
#### 4. 상태를 가진 인터페이스
```java
public interface ConnectionManager {
    // 상태를 나타내는 변수 (암묵적으로 public static final)
    ConnectionStatus STATUS = ConnectionStatus.DISCONNECTED;
    
    void connect();
    void disconnect();
}

// 구현 클래스
public class DatabaseConnection implements ConnectionManager {
    @Override
    public void connect() {
        // STATUS는 모든 구현체가 공유하는 상태가 됨 (의도하지 않은 결과)
        // STATUS = ConnectionStatus.CONNECTED; // 컴파일 오류! final이므로 변경 불가
    }
    
    @Override
    public void disconnect() {
        // 마찬가지로 STATUS 변경 불가
    }
}
```
#### 5. 클래스 계층 구조를 대체하는 인터페이스
```java
// 추상 클래스로 표현하는 것이 더 적절한 경우
public interface Animal {
    String getName();
    int getAge();
    void makeSound();
    void eat(Food food);
    void sleep();
    // 공통 기능 구현이 필요한 메서드들
}

// 많은 공통 코드 중복이 발생
public class Dog implements Animal {
    private String name;
    private int age;
    
    // 모든 Animal 구현체에서 반복되는 코드
    @Override
    public String getName() { return name; }
    
    @Override
    public int getAge() { return age; }
    
    // 실제로 다른 부분
    @Override
    public void makeSound() { System.out.println("Bark!"); }
    
    // 더 많은 중복 코드...
}
```
