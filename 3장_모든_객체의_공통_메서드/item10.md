# equals는 일반 규약을 지켜 재정의(Overriding)하라

*오버라이딩의 조건:

상위 클래스가 가지고 있는 멤버변수가 하위 클래스로 상속되는 것처럼 상위 클래스가 가지고 있는 메서드도 하위 클래스로 상속되어 하위 클래스에 사용할 수 있다. 또한 하위 클래스에서 메서드를 재정의해서도 사용할 수 있다. 쉽게 말해 메서드의 이름이 서로 같고, 매개변수가 같고, 반환형이 같을 경우에 상속받은 메서드를 덮어쓴다고 생각하면 된다. '부모 클래스의 메서드는 무시하고, 자식 클래스의 메서드 기능을 사용하겠다'와 같다.

1. Object의 equals() **(기본 구현)**

```
public boolean equals(Object obj) {
    return (this == obj); // 참조(메모리 주소) 비교
}
```

2. Thread의 equals() **(기본 구현)**

  - 예시 **(물리적 동치성)**
```
Thread t1 = new Thread();
Thread t2 = new Thread();
System.out.println(t1.equals(t2)); // false (다른 객체이므로)
System.out.println(t1 == t2); // false (참조가 다름)
```

3. Integer의 equals() **(오버라이딩)**
```
@Override
public boolean equals(Object obj) {
    if (obj instanceof Integer) {  // 객체가 Integer인지 확인
        return value == ((Integer) obj).value;  // 내부 값(value) 비교
    }
    return false;
}
```
  - 예시 **(논리적 동치성)**
```
Integer a = new Integer(100);
Integer b = new Integer(100);
System.out.println(a.equals(b)); // true (값이 같으므로)
```

### equals를 오버라이딩 하지 않아도 되는 경우

1. 각 인스턴스가 본질적으로 고유하다
> 값을 표현하는 클래스(`Integer`,`String`,`BigDecimal`)는 equals()를 값 비교 기준으로 구현한다. <br>
`Thread` 같은 동작하는 개체 클래스는 equals()가 객체의 동일성(참조 주소 비교)을 기준으로 구현된다. <br>
즉, Thread 클래스는 equals 메서드를 별도로 오버라이딩할 필요 없이, 객체의 참조를 비교하는 기본 동작이 적절하다.

2. 인스턴스의 논리적 동치성(logical equality)를 검사할 일이 없다.
> `Thread`같은 동작하는 개체(behavioral entity) 클래스들은 값이 아니라 개별적인 실행 흐름이 중요하다. 따라서 객체가 논리적으로 같은 지 비교할 일이 없다.

3. 상위 클래스에서 오버라이딩한 equals가 하위 클래스에도 들어맞는다.
> 대부분의 Set 구현체는 AbstractSet이 구현한 equals()를 상속받아 쓰고, List 구현체들은 AbstractList로부터,  Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다.

4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
> public의 경우는 어디서든지 호출이 가능하기 때문에 어떻게 쓰일지 예상할 수 없다. 대신 private나 package-private는 우리가 직접 equals()를 호출할 일이 없다면 equals() 메서드를 오버라이딩할 필요가 없다.

### equals를 overriding 하는 경우

> 물리적으로 같은지가 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다. 하지만 값 클래스라 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 재정의하지 않아도 된다. 인스턴스 통제 클래스는 특정 값에 대해 동일한 인스턴스가 하나만 존재하도록 보장한다. 따라서 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니 논리적 동치성과 객체 식별성이 사실상 같다.

### equals 메서드를 재정의할 때의 일반 규약

1. 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
```
public class Point {

	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
}
```

```
	Point point = new Point(1, 2);
	List<Point> points = new ArrayList<>();
	points.add(point);

	// equals()를 논리적 동치성을 판단하도록 오버라이딩해야 true다.
	points.contains(new Point(1, 2)); // false
```


> 객체는 자기 자신과 같아야 한다.
객체는 자기 자신과 같아야 한다는 뜻인데, 이 요건은 일부러 어기는 경우가 아니라면 만족시키지 못하기가 더 어렵다. 이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 false가 나오게된다.

2. 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
```
public final class CaseInsensitiveString {
	private final String s;

	public CaseInsensitiveString(String s) {
		this.s = Objects.requireNonNull(s);
	}

	// 대칭성 위배!
	@Override
	public boolean equals(Object o) {
		if (o instanceof CaseInsensitiveString)
			return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
		if (o instanceof String) // 한 방향으로만 작동한다!
			return s.equalsIgnoreCase((String) o);
		return false;
	}
}
```

```
		CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
		String s = "polish";

		cis.equals(s); // true
		s.equals(cis); // false
```
> HashSet에서 동일한 객체가 중복 저장될 수 있음 <br>
Map에서 get()이 예상과 다르게 동작할 수 있음 <br>
List.contains() 같은 메서드에서 특정 요소를 찾지 못할 가능성 있음

3. 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면 x.equals(z)도 true다.

```
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if(!(o instanceof Point)) // getclass 검사를 하라는 것은 아니다. (리스코프 치환 원칙 위배)
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
```

```
    //리스코프 치환 원칙 위배
    // 리스코프 치환 원칙: 프로그램에서 객체를 그 객체의 하위 타입으로 치환해도 프로그램의 올바른 동작이 유지되어야 한다.
    @Override public boolean equals(Object o){
      if(o == null || o.getClass() != getClass())
        return false;
      Point p = (Point) o;
      return p.x == x && p.y == y;
    }
```

```
public class ColorPoint extends Point {
	private final Color color;

	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}
	...
}
```

```
    ColorPoint p1 = new ColorPoint(1,2, Color.RED);
    Point p2 = new Point(1,2);
    ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
    p1.equals(p2);    // true
    p2.equals(p3);    // true
    p1.equals(p3);    // false
```

> 정렬 시 예상치 못한 결과 발생 <br>
TreeSet, TreeMap 같은 정렬 기반 컬렉션에서 요소가 이상하게 정렬되거나 사라지는 문제 발생 <br>
equals()를 기준으로 그룹핑할 때 일관성이 깨짐

4. 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true이거나 false다.

예를 들어 java.net.URL의 경우, equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교하도록 되어있다. 하지만 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하므로 그 결과가 항상 같다고 보장할 수 없다.

> HashMap, HashSet에서 동일한 객체가 들어갔다 나왔다 하는 등 이상한 동작 발생 <br>
equals()를 사용할 때 결과를 신뢰할 수 없어 코드가 오작동할 가능성 높음

5. null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

> 예상치 못한 NullPointerException 발생 가능 <br>
일부 API에서 null을 비교 대상으로 삼을 경우 오류 발생

### equals 메서드 구현 방법

1.  ==연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.

자기 자신이면 true를 반환한다.

2.  instanceof 연산자로 입력이 올바른 타입인지 확인한다.

가끔 클래스가 구현한 특정 인터페이스를 비교하는 경우가 있다. 이때, 인터페이스를 구현한 클래스라면 equals에서 (클래스가 아닌) 해당 인터페이스를 사용해야한다.

예를 들어 Set, List, Map, Map.Entry 등 컬렉션 인터페이스들이 이 경우에 해당한다.

3.  입력을 올바른 타입으로 형변환 한다.

2번에서 instanceof 연산자로 입력이 올바른 타입인지 검사를 했기에 이 단계는 100% 성공한다.

4.  입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

모두 일치해야 true를 반환한다. 2단계에서 인터페이스를 사용했다면, 입력의 필드 값을 가져올 때도 그 인터페이스의 메서드를 사용해야 한다.

```
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this) { //1번, 자기 자신 참조인지 확인
            return true;
        }

        if(!(o instanceof PhoneNumber)) { //2번, instanceof 확인
        	return false;
        }

        PhoneNumber pn = (PhoneNumber) o; //3번 형변환
        //4번 핵심 필드 일치하는지 확인
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```

### 주의사항
1. equals를 재정의 할 때는 hashcode도 반드시 재정의하자.
2. 너무 복잡하게 해결하려고 하지 말자. 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
3. Object 외의 타입을 매개변수로 받는 equals메서드를 정의하지 말자. 해당 메서드는 Object.equals를 오버라이드 한 게 아니라 오버로딩 한 것에 불과하다.