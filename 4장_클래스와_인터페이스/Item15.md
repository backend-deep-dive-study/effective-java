# 클래스와 멤버의 접근 권한을 최소화 하라

> 정보 은닉(캡슐화)를 통해 유지보수성과 보안성을 높이고, 코드의 복잡도를 낮춰라

## 캡슐화

> 클래스 내부 구현이 외부에 노출되면, 한 클래스의 변경이 다른 여러 클래스에 영향을 미침 → 변경할 때마다 수정해야 할 코드가 많아짐!

```
class User {
    public String name;
    public int age;
}
```

```
class User {
    private String name;
    private int age;

    public String getName() {
	return name;
    }
    public int getAge() {
	return age;
    }
}
```

```
class Service {
    public void printUserInfo(User user) {
        // user의 이름 뒤에 "님"을 붙이고 싶다면? user.name을 쓰는 곳을 찾아 +"님" 을 해야한다.
	System.out.println(user.name);
	// getName메서드에서 로직을 수정하면 외부 코드에 영향이 없다.
	System.out.println(user.getName());
    }
}
```

- 시스템 개발 속도를 높인다. 
→ 여러 컴포넌트를 병렬로 개발
- 시스템 관리 비용을 낮춘다. 
→ 각 컴포넌트를 빨리 파악하여 디버깅 & 다른 컴포넌트로 교체하는 부담도 적음
- 정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다. 
→ 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문
- 소프트웨어 재사용성을 높인다. 
→ 외부에 거의 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면 그 컴포넌트와 함께 개발되지 않은 낯선 환경에 
서도 유용하게 쓰일 가능성 이 크기 때문
- 큰 시스템을 제작하는 난이도를 낮춰준다.
→ 시스템 전체가 아직 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있기 때문이다.


## 접근 제한자

### 종류
> 멤버（필드, 메서드, 중첩 클래스, 중첩 인터페이스）에 부여할 수 있는 접근 수준
- private : 멤버를 선언한 톱레벨 클래스에서 접근 가능, 멤버에만 사용 가능, 비공개 API
- package-private : 멤버가 소속된 패키지 안의 모든 클래스에 접근 가능, 비공개 API
한 클래스에서만 사용하는 클래스는 private static으로 중첩 클래스를 사용)
- protected : 동일 패키지 혹은 하위 클래에서 접근 가능, 멤버에만 사용 가능, 공개 API
- public : 모든 곳에서 접근 가능, 공개 API

> 기본 원칙 : 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다. <br>
톱레벨 클래스와 인터페이스에 부여할 수 있는 접근 수준은 package-private과 public 두 가지다. <br>
톱레벨 클래스나 인터페이스를 public으로 선언하면 공개 API가 되며, package-private으로 선언하면 해당 패키지 안에서만 이용할 수 있다. 패키지 외부에서 쓸 이유가 없다면 package-private 으로 선언하자. 그러면 이들은 API가 아닌 내부 구현이 되어 언제든 수정할 수 있다.


- 클래스의 공개 API를 세심히 설계한 후 모든 멤버를 private로 만든다. 이후 같은 패키즈의 다른 클래스가 접근해야 하는 멤버는 package-private로 풀어준다.
→ 만약 권한을 풀어주는 일이 자주 일어난다면 설계를 고민해보자!

- Serializable을 구현한 클래스는 필드들이 public이 아니더라도 의도치 않게 공개 API가 될 수 있으므로 주의하자

- public 클래스의 멤버 접근 수준이 protected 이상이라면 멤버에 접근할 수 있는 대상 범위가 매우 늘어난다. 공개 API로 간주하고 영원히 지원되야 함을 명심하자.

- 테스트를 위해 접근제어자를 풀어주는 건 어느정도 허용되지만, public으로 만드는 것은 안 된다. 

- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.  클래스의 내부 구현 세부사항을 외부에 노출시키지 않고, 외부에서 직접적으로 접근하여 변경하지 못하도록 하는 것을 의미한다. 가능한한 클래스의 인스턴스 필드는 private으로 선언하고, 필요한 경우에만 접근자(getter)와 설정자(setter)를 제공하여 외부에서 간접적으로 접근하도록 하는 것이 바람직하다. 이를 통해 정보 은닉과 캡슐화를 유지하고, 안정성과 유연성을 높여 데이터 유효성을 보장해준다.

- public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다. 멀티 스레드 환경에서 해당 클래스의 인스턴스를 여 러 스레드가 동시에 접근하고 수정할 때 문제가 발생할 수 있다. 예를 들어, 한 스레드가 값을 읽는 동안 다른 스레드가 값을 변경할 수 있으며, 이로 인해 데이터 불일치가 발생할 수 있다.

- 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다. 

public : 모든 클래스에서 접근 할 수 있다. <br>
static :  클래스의 인스턴스화 없이 사용할 수 있다. <br>
final : 한 번 초기화되면 그 값이 변경되지 않는다. 즉, 해당 필드는 상수로 취급된다. (상수 : 고정된 값)

아래의 클래스는 public static final 배열 필드를 사용하고, 해당 필드를 반환하는 접근자 메서드를 제공한다. 보안 허점이 숨어있는 코드이다.
```
// 불변성 보장X, 캡슐화 원칙 위배
public class Constants {
    public static final String[] COLORS = {"Red", "Green", "Blue"};

    public static String[] getColors() {
        return COLORS;
    }
}
```

해결 방법
```
public class Constants {
    private static final String[] COLORS = {"Red", "Green", "Blue"};

    // getColors 메서드: COLORS 배열을 변경할 수 없는 리스트 형태로 반환
    public static List<String> getColors() {
        // Arrays.asList 메서드를 사용하여 COLORS 배열을 리스트로 변환
        // 이 메서드는 고정 크기의 리스트를 반환하므로 리스트의 크기는 변경될 수 없음
        // unmodifiableList 메서드를 사용하여 리스트를 변경할 수 없는 리스트로 래핑
        return Collections.unmodifiableList(Arrays.asList(COLORS));
    }
}
```


## 모듈

> 패키지들의 묶음

- 사용 예시 : JDK
	- java.base, java.sql, java.xml와 같은 모듈을 활용하여 필요한 패키지만 선택적으로 사용하여 애플리케이션의 크기와 복잡도를 줄일 수 있음
- 모듈은 자신이 속한 패키지 중 공개할 것을 선언(module-info.java)하는데 protected, public 멤버라 할 지라도 외부에서는 접근이 불가
- 모듈 내부에서는 자유롭게 멤버를 사용 가능
- 모듈의 JAR 파일을 모듈 경로가 아닌 애플리케이션 클래스패스에 두면 모듈이 없는 것처럼 행동하므로 주의
	- 클래스패스(--class-path 또는 -cp) : 기존 클래스 방식으로 처리
	- 모듈 경로(--module-path) : 모듈 시스템 규칙으로 처리
### 단점
- 패키지들을 모듈 단위로 묶고 모듈 선언에 패키지들의 모든 의존성 명시
- 소스트리 재배치
- 모듈 안으로부터 일반 패키지로의 모든 접근에 특별한 조치 필요
- exports: 모듈 외부에 특정 패키지를 공개하여 다른 모듈에서 접근
- opens: 리플렉션을 통해 특정 패키지에 접근할 수 있도록 허용

> 꼭 필요한 경우가 아니라면 사용 비권장
