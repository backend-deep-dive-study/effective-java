# 자바 직렬화의 대안을 찾으라

- 직렬화 

    > 객체 직렬화란 자바가 객체를 바이트 스트림으로 인코딩하고(직렬화)그 바이트 스트림으로부터 다시 객체를 재구성하는(역직렬화) 메커니즘이다. 직렬화된 객체는 다른 VM에 전송하거나 디스크에 저장한 후 나중에 역직렬화할 수 있다.

## 직렬화의 문제

- `Objectinputstream`

    >  `readObject` 메서드는 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있는 생성자다. <br>
    바이트 스트림을 역직렬화하는 과정에서 이 메서드는 그 타입들 안의 모든 코드를 수행할 수 있다. <br>
    즉, 그 타입들의 코드 전체가 공격 범위에 들어간다는 뜻이다.

    ```
    import java.io.IOException;
    import java.io.ObjectInputStream;
    import java.io.Serializable;

    public class EvilObject implements Serializable {
        private static final long serialVersionUID = 1L;

        private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
            Runtime.getRuntime().exec("calc");  // 윈도우 계산기 실행 (예시용 악성코드)
        }
    }
    ```
- 가젯(Gadget) : 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드

    >  여러 가젯을 함께 사용하여 가젯 체인을 구성할 수도 있는데, 가끔씩 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있는 아주 강력한 가젯 체인도 발견되곤 한다. <br>
    샌프란시스코 교통국을 마비시킨 공격이 정확히 이런 사례로, 가젯들이 체인으로 엮여 피해가 더욱 컸다.

- 역직렬화 폭탄(deserialization bomb) : 역직렬화에 시간이 오래 걸리는 짧은 스트림. 서비스 거부 공격(denial-of-service, DoS)의 일종.

    > 아래 객체 그래프는 201개의 HashSet 인스턴스로 구성되며, 그 각각은 3개 이하의 객체 참조를 갖는다. <br>
    스트림의 전체 크기는 5744바이트지만, 역직렬화는 태양이 불타 식을 때까지도 끝나지 않을 것이다.

    ```
    static byte[] bomb() {
        Set<Object> root = new HashSet<>();
        Set<Object> s1 = root;
        Set<Object> s2 = new HashSet<>();
        for (int i = 0; i < 100; i++) {
            Set<Object> t1 = new HashSet<>();
            Set<Object> t2 = new HashSet<>();
            t1.add("foo"); // t1을 t2와 다르게 만든다. 
            s1.add(t1);
            s1.add(t2);
            s2.add(t1);
            s2.add(t2);
        }
        return serialize(root); // 간결하게 하기 위해 이 메서드의 코드는 생략함
    }
    ```
## 해결 방법

- 직렬화 위험을 회피하는 가장 좋은 방법은 아무것도 역직렬화하지 않는 것이다.

    > 자바 직렬화를 써야 할 이유는 전혀 없다. 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘이 많이 있다. <br>
    이 방식들은 자바 직렬화의 여러 위험을 회피하면서 다양한 플랫폼 지원, 우수한 성능, 풍부한 지원 도구, 활발한 커뮤니티와 전문가 집단 등 수많은 이점까지 제공한다. <br>
    이런 메커니즘들도 직렬화 시스템이라고 불리지만, 자바 직렬화와 구분하고자 `크로스-플랫폼 구조화된 데이터 표현(Cross-platform structured-data representation)` 이라 한다.
    
    - 예시 : JSON, XML, Protocol Buffers, MessagePack, Kryo

- 레거시 시스템 때문에 자바 직렬화를 완전히 배제할 수 없을 때의 차선책은 신뢰할 수 없는 데이터는 절대 역직렬화하지 않는 것이다.

    > 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없다면 객체 역직렬화 필터링(`java.io.ObjectInputFilter`)을 사용하자 <br>
    자바 9에 추가되었고, 이전 버전에서도 쓸 수 있도록 이식되었다. <br>
    객체 역직렬화 필터링은 데이터 스트림이 역직렬화되기 전에 필터를 설치하는 기능이다. 클래스 단위로, 특정 클래스를 받아들이거나 거부할 수 있다.

## 결론

> 직렬화는 위험하니 피해야 한다. → 시스템을 밑바닥부터 설계한다면 JSON이나 프로토콜 버퍼 같은 대안을 사용하자. <br>
> 신뢰할 수 없는 데이터는 역직렬화하지 말자. → 꼭 해야 한다면 객체 역직렬화 필터링을 사용하되, 이마저도 모든 공격을 막아줄 수는 없음. <br>
> 자바 직렬화를 사용하는 시스템을 관리해야 한다면 시간과 노력을 들여서라도 `크로스-플랫폼 구조화된 데이터 표현`으로 마이그레이션하는 것을 심각하게 고민해봐야 한다.
