# 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

- 싱글턴 패턴을 사용해 인스턴스를 하나로 제한할 수 있다.
    ```java
    public class Elvis {
        public static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }
        public void leaveTheBuilding() { ... }
    }
    ````
- 아이템 3에서 얘기했듯이 Elvis 클래스에 `Serializable`을 구현하면 역직렬화 시 새로운 인스턴스가 생성되어 싱글턴이 아니게 된다.

    - 기본 직렬화(아이템 87), 명시적인 readObject(아이템 88)도 소용없다.

## readResolve()

- `readResolve()`는 역직렬화된 인스턴스를 다른 인스턴스로 바꾸는 메서드

- Elvis 클래스가 `Serializable`을 구현할 때, readResolve 메서드를 추가해 싱글턴을 유지할 수 있다.

    ```java
    private Object readResolve() {
        return INSTANCE;
    }
    ```

- 이 메서드는 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis 인스턴스를 반환한다.

    - 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다.

    - 대부분의 경우 이때 새로 생성된 객체의 참조는 유지 하지 않으므로 바로 가비지 컬렉션 대상이 된다.

- **readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴 스 필드는 모두 transient로 선언해야 한다.**

    - transient가 아닌 참조 필드가 있다면 해당 필드는 역직렬화 시 자동으로 복원된다.

    - trasient가 아닌 참조 필드가 있따면 MutuablePeriod 공격과 유사한 방식으로 공격할 여지가 남는다.

### 공격 예시 - 도둑(stealer) 클래스

- 아이디어

    1. 싱글턴이 non-transient 참조 필드를 가지고 있다.

    2. 그럼 해당 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화된다.

    3. 그렇다면 잘 조작된 스트림을 써서 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}

public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
        //resolve되기 전의 Elvis 인스턴스의 참조를 저장한다.
        impersonator = payload;

        // favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
        return new String[] { "A Fool Such as I" };
    }
    private static final long serialVersionUID = 0;
}

public class Elvislmpersonator {
    // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림!
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
        0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
        (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
        0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
        0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
        0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
        0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
        0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0X0C, 0x45, 0x6c, 0x76,
        0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0X00,
        0X00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0X01,
        0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
        0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
        0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
    };
    
    public static void main(String[] args) {
        // ElvisStealer.impersonator를 초기화한 다음,
        // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
        Elvis elvis = (Elvis) deserializer(serializedFom);
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites();
    }

    // 실행결과
    // > [Hound Dog, Heartbreak Hotel]
    // > [A Fool Such as I]
```

- 자세한 공격 방식

    1. 먼저, readResolve 메서드와 인스턴스 필드 하나를 포함한 `도둑(stealer) 클래스`를 작성한다.
        - 이 인스턴스 필드(impersonator)는 도둑이 ‘숨길’ 직렬 화된 싱글턴을 참조하는 역할을 한다.

    2. 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 이 도둑의 인스턴스로 교체한다.

    3. 이제 싱글턴은 도둑을 참조하고 도 둑은 싱글턴을 참조하는 순환고리가 만들어졌다.

    4. 싱글턴이 도둑을 포함하므로 싱글턴이 역직렬화될 때 도둑의 readResolve 메서드가 먼저 호출된다.

    5. 도둑의 readResolve 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인 (그리고 readResolve가 수행되기 전 인) 싱글턴의 참조가 담겨 있게 된다.

    6. 도둑의 readResolve 메서드는 이 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다.

    7. 이 메서드는 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.
        - 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 `ClassCastException`을 던진다.

### 해결책 - 열거 타입 싱글턴

```java
public enum Elvis {
    INSTANCE;

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        System.out.println(Arrays.toString(fa\/oriteSongs));
    }
}
```

- 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언 한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.

- 물론 공격자가 `AccessibleObject.setAccessible` 같은 특권(privileged) 메서드를 악용한다면 이야기가 달라진다.
    - 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 모든 방어가 무력화된다.

## 마무리

- 인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다.

    - 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문이다.

- **readResolve 메서드의 접근성은 매우 중요하다.**

    - final 클래스이라면 readResolve 메서드는 private이어야 한다.

    - final 클래스가 아니라면 몇 가지 주의 사항이 있다.

        - private으로 선언하면 하위 클래스에서 사용할 수 없다.

        - package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.

        - protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
            - protected나 public이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스르 생성하여 `ClassCastException`을 일으킬 수 있다.

## 결론

- 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자.

- 직렬화와 인스턴스 통제가 모두 필요하다면 readResolve 메서드를 작성해 넣어야 하고, 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야 한다.