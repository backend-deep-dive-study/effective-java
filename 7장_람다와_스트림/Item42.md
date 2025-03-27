# 익명 클래스보다는 람다를 사용하라
## 함수 객체의 과거 방식 : 익명클래스
- 자바 8 이전에는 함수 객체를 만들기 위해 주로 익명 클래스를 사용
- 너무 장황하고 가독성이 떨어짐
```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```
## 자바 8 이후 : 람다 도입
- 자바 8부터는 함수형 인터페이스를 람다로 간결하게 구현 가능
- 간결함과 명확성이 뛰어남
- 타입 추론을 통해 대부분의 타입은 생략 가능 
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
### 람다의 타입 추론
- 컴파일러가 문맥을 보고 타입을 추론해줌
- 제네릭을 사용할 때 람다에서 타입 추론이 더 효과적으로 작동
- 타입을 명시해야 할 때를 제외하고는 람다의 모든 매개변수 타입은 생략하는게 좋음
### 람다 응용 : 더 간단한 표현
- Comparator의 정적 메서드 사용
```java
Collections.sort(words, comparingInt(String::length)); // Comparator 인터페이스에 comparingInt가 정의되어 있음
```
- List의 sort 메서드 이용
```java
words.sort(comparingInt(String::length));
```
### 람다 활용 에시
- 열거 타입에서 상수별 동작을 구현할 때 람다를 활용할 수 있음
```java
public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    @Override public String toString() { return symbol; }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```
## 람다의 제약사항
- 이름이 없고 문서화 불가능
- 길고 복잡한 로직은 람다로 표현하지 않는 것이 좋음(3줄 이내가 이상적)
- 함수형 인터페이스에서만 사용 가능(추상 메서드가 하나인 인터페이스)
- 람다에서 this는 람다를 감싸는 인스턴스를 가리킴
- 자신을 참조해야 할 때는 익명 클래스 사용 필요
- 직렬화 형태가 구현별로 다를 수 있어 직렬화는 피해야 함
## 익명 클래스가 필요한 경우
- 추상 클래스의 인스턴스를 만들 때
- 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때
- 함수 객체가 자신을 참조해야 할 때

## 정리
- 자바 8부터 작은 함수 객체를 구현하는 데 적합한 람다 도입
- 익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용할 것

## 부록
### 함수 객체란?
- 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스의 인스턴스
### 함수형 인터페이스란?
- 정확히 하나의 추상 메서드만을 가진 인터페이스를 의미. 자바 8에서 람다 표현식을 지원하기 위해 도입된 개념!
```java
@FunctionalInterface
public interface Comparator<T> {
    // 단 하나의 추상 메서드
    int compare(T o1, T o2);
    
    // 디폴트 메서드 (여러 개 가능)
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    
    // 정적 메서드 (여러 개 가능)
    static <T> Comparator<T> naturalOrder() {
        return (T o1, T o2) -> ((Comparable<T>)o1).compareTo(o2);
    }
}
```
