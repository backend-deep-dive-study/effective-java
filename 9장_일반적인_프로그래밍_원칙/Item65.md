# 리플렉션보다는 인터페이스를 사용하라

### 리플렉션

> 리플렉션 기능을 이용하면 임의의 클래스에 접근 가능 <br>
Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있고,<br>
이어서 이 인스턴스들로는 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져 
올 수 있다. <br> 
나아가 Constructor, Method, Field 인스턴스를 이용해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있다.

- 구체적인 클래스 타입을 알지 못해도 그 클래스의 정보(메서드, 타입, 변수 등등)에 접근할 수 있게 해주는 자바 API

```java
import java.lang.reflect.Field;
import java.lang.reflect.Method;

class Person {
    private String name; //접근제한 -> 리플렉션 없이는 접근 및 수정 불가
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void sayHello() {
        System.out.println("Hello, my name is " + name + " and I am " + age + " years old.");
    }
}

public class ReflectionExample {
    public static void main(String[] args) {
        try {
            // Person 클래스의 인스턴스 생성
            Person person = new Person("John", 25);

            Class<?> personClass = person.getClass();

            // 필드에 접근하여 값 변경하기 (필드가 private, protected, public 이든 상관없음)
            Field nameField = personClass.getDeclaredField("name"); // 리플렉션을 사용한 필드 접근
            nameField.setAccessible(true);  // private 필드에 접근 가능하도록 설정
            nameField.set(person, "Jane");

            // 메서드 호출하기
            Method sayHelloMethod = personClass.getDeclaredMethod("sayHello");
            sayHelloMethod.invoke(person);  // "Hello, my name is Jane and I am 25 years old." 출력

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 리플렉션의 단점

- 컴파일 타입 검사의 이점을 누릴 수 없다.
    - 런타임에야 오류를 알게될 것이다.
- 코드가 지저분해진다.
- 성능이 떨어진다.
    - 리플렉션을 통한 메서드 호출은 상당히 느리다.

> 일반적인 코드에서는 리플렉션이 필요 없다. 코드 분석 도구나 의존관계 주입 프레임워크와 같은 복잡한 애플리케이션에서나 리플렉션이 가끔 필요하다. <br>
리플렉션은 아주 제한된 형태로만 사용해야 단점을 피하고 이점만 취할 수 있다.

### 리플렉션 예시 코드
> 이는 제네릭 집합 테스트의 역할을 한다.

```java
import java.util.*;
import java.lang.reflect.*;

public class Item65Test {
    public static void main(String[] args) {
        Item65Test test = new Item65Test();
        test.reflectionTest();
    }

    public void reflectionTest() {
        // 클래스 가져오기
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>) Class.forName("java.util.LinkedHashSet");
        } catch (ClassNotFoundException e) {
            // 해당 경로에서 클래스를 찾을 수 없을 때
            e.printStackTrace();
        }

        // 생성자 얻기
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            // 매개변수 없는 생성자가 없을 때
            e.printStackTrace();
        }

        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (InvocationTargetException e) {
            // 생성자가 예외를 던졌을 때
            e.printStackTrace();
        } catch (InstantiationException e) {
            // 클래스를 인스턴스화할 수 없을 때
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            // 생성자에 접근할 수 없을 때
            e.printStackTrace();
        } catch (ClassCastException e) {
            // Set을 구현하지 않은 클래스일 때
            e.printStackTrace();
        }

        s.addAll(List.of("안녕", "안녕", "하세", "요", "."));

        System.out.println("s = " + s);

        StringJoiner sj = new StringJoiner("");

        for (String s1 : s) {
            sj.add(s1);
        }

        String s1 = sj.toString();

        System.out.println("s1 = " + s1);
    }
}
```

> 클래스 경로 문자열을 통하여 해당 클래스를 가져오고, 가져온 클래스를 조작할 수 있다. <br>
위 코드는 Set을 통하여 문자열을 다뤄봤다. <br.>
LinkedHashSet과 Set을 테스트해봤다. <br>

- java.util.LinkedHashSet의 결과

s = [안녕, 하세, 요, .]

s1 = 안녕하세요.

순서를 지킨다.
- java.util.HashSet의 결과

s = [요, 안녕, 하세, .]

s1 = 요안녕하세.

순서를 지키지 않는다.

### 예제에서 보는 리플렉션의 단점

> 런타임에 6가지 예외가 발생 가능하다. <br>
제대로 예외처리를 한다면 고려할 점이 너무 많을 수 있다. <br>
다만, 리플렉션 예외를 잡는데는 자바7부터 ReflectiveOperationException 하나로 가능하긴 하다. <br>
클래스 이름만으로 클래스를 생성하기 위해서는 25줄의 코드를 작성해야 한다. <br>
일반 자바코드를 쓰는 것보다 압도적으로 코드가 많다.

### 결론

>리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다. <br>
컴파일 타임에는 알 수 없는 클래스를 사용하는 프로그램을 작성하면 리플렉션을 사용해야 한다. <br>
되도록, 객체 생성에만 사용하고 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용하자.