#  ordinal 메서드 대신 인스턴스 필드를 사용하라

> 대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 그리고 모 든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 
ordinal이라는 메서드를 제공한다.

```
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, 
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

### 문제점

> 상수 선언 순서에 의존: 열거 타입 상수의 선언 순서를 바꾸면 ordinal() 값도 바뀌어 예상치 못한 동작을 할 수 있습니다. <br>
중복된 값 추가 불가: 만약 동일한 값을 갖는 새로운 상수를 추가해야 한다면 ordinal()을 사용할 수 없습니다.

### 해결책

> 열거 타입 상수에 연결된 값은, ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하자.

<details>
  <summary>인스턴스 필드</summary>

: 클래스나 열거 타입의 각 인스턴스가 고유하게 가지는 변수    
</details>


```
public enum Ensemble {
    // 인스턴스 필드에 정수 데이터를 저장하는 열거 타입
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians; // 인스턴스필드
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

### 결론

> ordinal 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다. 따라서 이런 용도가 아니라면 ordinal 메서드를 절대 사용하지 말자.