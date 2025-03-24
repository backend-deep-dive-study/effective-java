# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### 타입 안전 열거 패턴 vs 열거 타입

- 타입 안전 열거 패턴은 확장할 수 있음 -> 열거한 값 + 다음 값을 더 추가해 다른 목적으로 사용할 수 있음
- 열거 타입은 확장할 수 없음
- 만약 열거 타입을 확장한다면?
  - 확장한 타입이 원소는 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않음
  - 기반 타입과 확장된 타입 원소 모두를 순회할 방법이 마땅하지 않음
  - 확장성을 높이기 위해 고려할 요소가 늘어나 설계와 구현이 복잡해짐

### 확장할 수 있는 열거 타입의 쓰임새

- 연산 코드 (operation code, opcode)
- 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻함
- API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가해야 할 때 필요
- 연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하도록 할 수 있음

### 예시

```java
public interface Operation {
  double apply(double x, double y);
}
```

- Operation 인터페이스를 구현한 기본 연산

```java
public enum BasicOperation implements Operation {
  PLUS("+") {
    public double apply(double x, double y) {
      return x + y;
    }
  }
  ...
}
```

- 추가한 연산

```java
public enum ExtendedOperation implements Operation {
  EXP("^") {
    public double apply(double x, double y) {
      return Math.pow(x, y);
    }
  }
  ...
}
```

### 두 연산을 모두 활용하는 방법

1. 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용하게 하는 방법

```java
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation>
// Class 리터럴이 한정적 타입 토큰 역할을 함
void test(Class<T> opEnumType, double x, double y) {
  for (Operation op : opEnumType.getEnumConstants()) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```

=> Class 객체는 열거 타입인 동시에 Operation의 하위 타입이어야 함

2. Class 객체 대신 한정적 와일드카드 타입인 `Collection<? extends Operaiotn>`을 넘기는 방법

```java
public static void main(String[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for (Operation op : opSet) {
    System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
  }
}
```
- 코드가 덜 복잡하고 여러 구현 타입의 연산을 조합해 호출할 수 있음
- 문제점 : 열거 타입끼리 구현을 상속할 수 없음

## 결론
- 열거 타입은 확장할 수 없음
- 인터페이스 + 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 확장한 효과를 낼 수 있음
- 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있음!
