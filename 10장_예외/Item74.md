# 메서드가 던지는 모든 예외를 문서화하라

- 메서드가 던지는 예외 하나하나를 문서화하는 데 충분히 시간을 쏟아야 한다.

- 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 @throws 태그를 사용하여 정확히 문서화하자.

- 메서드가 사용자에게 각 예외에 대처할 수 있는 힌트를 주어야 한다.

    - 따라서 절대로 Exception이나 Throwable을 던진다고 선언해서는 안된다.

    - 단, main에서는 오직 JVM만이 호출하므로 Exception을 던지도록 선언해도 괜찮다.

## 비검사 예외도 검사 예외처럼 문서화해두면 좋다.

- 비검사 예외는 일반적으로 프로그래밍 오류를 뜻하기에, 일으킬 수 있는 오류들이 무엇인지 알려주면 프로그래머는 자연스럽게 해당 오류를 피하게 코딩하게 된다.

- 특히나 인터페이스 메서드에서 특히 비검사 예외를 문서로 남기는 것은 중요하다.

    - 이 조건이 인터페이스의 일반 규약에 속하게 되어 그 인터페이스를 구현한 모든 구현체가 일관되게 동작하도록 해주기 때문이다.

- 메서드가 던질 수 있는 예외를 각각 @throws 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.

    - 검사냐 비검사에 따라 API 사용자가 해야 할 일이 달라지므로 확실히 구분하는게 좋다.

- 비검사 예외를 모두 문서화하는 것은 현실적으로 불가능할 때도 있다.

- 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그 예외를 (각각의 메서드가 아닌) 클래스 설명에 추가하는 방법도 있다.

    - NullPointerException이 가장 흔한 사례로, "이 클래스의 모든 메서드는 인수로 null이 넘어오면 NullPointerException을 던지다'라고 적어야 한다.