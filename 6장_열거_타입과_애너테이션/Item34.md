# int 상수 대신 열거 타입을 사용하라

- 열거 타입은 일정 개수의 상수 값을 정의하고, 그 외의 값은 허용하지 않는 타입이다.
  - 예) 사계절, 태양계의 행성, 카드게임의 카드 종류, ...

## 열거 타입 이전의 방식 - 정수 열거 패턴(int enum pattern)

- 열거 타입을 지원하기 전에는 정수 상수를 한 묶음 선언하는 정수 열거 패턴(int enum pattern) 기법을 사용했다.

```java
public static final int APPLE_FUJI         = 0;
public static final int APPLE_PIPPIN       = 1;
public static final int APPLE_GRANNY_SMITY = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD  = 2;
```

### 단점

1.  타입 안전을 보장할 방법이 없으며 표현력이 좋지 않다.

2. 연관이 없는 상수로 동등 연산자(==) 비교하더라도 경고 메시지가 나지 않는다.

  ```java
  int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
  ```

3. 이름을 사용해서 충돌을 방지해야 한다.

  - 수은(원소)과 수성(행성)을 구분하기 위해서 ELEMENT_MERCURY와 PLANET_MERCURY로 지어 구분해야 한다.

4. 깨지기 쉽다.

  - 컴파일 시 상수 값들이 클라이언트 파일에 그대로 새겨진다.(하드코딩됨)

    ```java
    if (fruit == APPLE_FUJI) { // 컴파일 시 'if (fruit == 0)'로 변환됨 
            System.out.println("후지 사과입니다");
    }
    ```
  
  - 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.

  - 컴파일하지 않는다면 실행이 되더라도 의도한대로 엉뚱하게 동작할 것이다.

5. 문자열로 출력하기가 까다롭다.

  - 디버깅을 하면 단지 숫자로만 보여서 도움이 되지 않는다.

  - 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 것도 좋지 않다.(상수가 몇 개인지도 알 수 없다.)

## 문자열 열거 패턴(string enum pattern)

```java
public static final int APPLE_FUJI         = "fuji";
public static final int APPLE_PIPPIN       = "pippin";
public static final int APPLE_GRANNY_SMITY = "granny_smity";
```

- 정수 열거 패턴보다 더 나쁘다.

- 상수의 의미를 출력할 수 있다는 점은 좋지만, 문자열 값을 하드코딩해야 한다.

- 오타가 있어도 컴파일러가 확인할 수 없으니 자연스럽게 런타임 버그가 생긴다.

- 문자열 비교에 따른 성능 저하도 생긴다.

## 열거 타입(enum type)

- 열거 패턴의 단점을 말끔히 씻어주며 여러 장점이 더해진 대안이다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

- 자바의 열거 타입은 완전한 형태의 클래스랏 다른 언어의 열거 타입보다 훨씬 강력하다.

### 장점

1. 인스턴스 통제된다.
  - 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.

  - 또한, 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.

  - 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩 존재한다.

  - 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

2. 컴파일타임 타입 안전성을 제공한다.

  - Apple의 열거타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 (null이 아니라면) Apple의 세 가지 값 중 하나임이 확실하다.

3. 열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.

4. 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.

  - 공개되는 것이 오직 필드의 이름뿐이라, 상수 값이 클라이언트로 컴파일되어 각인되지 않는다.

5. 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.

6. 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.

### 상수와 서로 다른 데이터 연결하기

- 태양계의 여덟 행성의 질량과 반지름, 그리고 표면중력을 가지는 경우

  ```java
  public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;    // F = ma
    }
  }
  ```

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

- 열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다.

- 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는 게 낫다.

- 열거 타입에서 상수를 하나 제거하더라도 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.

  - 설사 참조한 클라이언트에서도 컴파일 시에 컴파일 오류가 발생한다.

  - 컴파일을 하지 않더라도 런타임에 예외가 발생한다.

### 상수마다 서로 다른 동작하기

- 사칙연산 계산기의 연산 종류에 따라 연산 수행을 하는 경우

  - 나쁜 예

    ```java
    public enum Operation {
      PLUS, MINUS, TIMES, DIVIDE;

      public double apply(double x, double y) {
        switch(this) {
          case PLUS:   return x + y;
          case MINUS:  return x - y;
          case TIMES:  return x * y;
          case DIVEDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: + this);
      }
    }
    ```

    - 새로운 상수를 추가하면 case문도 추가해야 한다.

  - 좋은 예

    - 열거 타입에 추상 메서드를 선언하고 각 상수별 클래스 몸체에서 재정의하기.

      ```java
      public enum Operation {
        PLUS  {public double apply(double x, double y){return x + y;}},
        MINUS {public double apply(double x, double y){return x - y;}},
        TIMES {public double apply(double x, double y){return x * y;}},
        DIVIDE{public double apply(double x, double y){return x / y;}};

        public abstract double apply(double x, double y);
      }
      ```
  
    - toString을 재정의해 해당 연산을 뜻하는 기호를 반환하기.

      ```java
      public enum Operation {
        PLUS("+") {
            public double apply(double x, double y) { return x + y; }
        },
        MINUS("-") {
            public double apply(double x, double y) { return x - y; }
        },
        TIMES("*") {
            public double apply(double x, double y) { return x * y; }
        },
        DIVIDE("/") {
            public double apply(double x, double y) { return x / y; }
        };

        private final String symbol;

        Operation(String symbol) { this.symbol = symbol; }

        @Override public String toString() { return symbol; }
        public abstract double apply(double x, double y);
      }
      ```

### toString을 재정의하려거든, fromString 메서드 제공을 고려해보라

- 열거 타입에는 상수 이름을 입력받아 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.

  ```java
  Operation o = Operation.valueOf("PLUS"); // Operation.PLUS 반환환
  ```

- toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 제공하는 걸 고려해보자.

  ```java
  private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(
            toMap(Object::toString, e -> e));

  // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
  public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
  }
  ```

  - Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다.

  - Optional<Operation>인 이유는 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 클라이언트에 알려 대처하도록 것이다.

### 전략 열거 타입 패턴

- 전략을 사용해 상수별 메서드의 중복을 줄일 수 있다.

- 나쁜 예

  ```java
  enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;
            default: // 주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
  }
  ```
  - 관리 관점에서 위험한 코드다.
  
    - 휴가와 같은 새로운 값을 추가하려면 case문에도 넣어줘야 한다.

- 좋은 예

  ```java
  enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
      return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
      WEEKDAY {
        int overtimePay(int minsWorked, int payRate) {
          return minsWorked <= MINS_PER_SHIFT ? 0 :
            (minsWorked - MINS_PER_SHIFT) * payRate / 2;
        }
      },
      WEEKEND {
        int overtimePay(int minsWorked, int payRate) {
          return minsWorked * payRate / 2;
            }
      };

      abstract int overtimePay(int mins, int payRate);
      private static final int MINS_PER_SHIFT = 8 * 60;

      int pay(int minsWorked, int payRate) {
        int basePay = minsWorked * payRate;
        return basePay + overtimePay(minsWorked, payRate);
      }
    }
  }
  ```

### 열거 타입의 상수별 동작의 혼합

- switch문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않음을 많이 봐왔다.

- 하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.

```java
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
        default: throw new AssertionError("알 수 없는 연산: " + op);
    }
}
```

## 결론

- 대부분의 경우 열거 타입의 성능은 정수 상수와 별반 다르지 않다.

- 열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들긴 하지만 체감될 정도는 아니다.

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.

  - 태양계 행성, 한 주의 요일, 체스 말, 메뉴 아이템 등

- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.