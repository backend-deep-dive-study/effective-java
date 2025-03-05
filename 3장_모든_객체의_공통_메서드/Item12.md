## toString을 항상 재정의하라
### Object의 기본 toString 메서드
- 기본 toString은 **"클래스_이름@16진수로_표시한_해시코드"형태**로 반환
- 객체에 유익한 정보를 반환하지 못함.
### toString 재정의의 이점
- 재정의를 통해 toString은 간결하고 사람이 읽기 쉬운 형태로 정보를 제공할 수 있음.
- 클래스 사용과 시스템 디버깅이 쉬워짐.
- 로깅, 디버깅, 오류 메시지 등에서 자동으로 호출될 때 유용한 정보를 제공함.
- 컬렉션 내부의 객체 표현도 개선됨. -> List, Map, Set 등
- 데이터 검증이 보다 용이해질 수 있음.
### toString은 그 객체가 가진 주요 정보 모두를 반환시키자
- 효과적인 toString을 구현하기 위해선 객체의 주요 정보를 모두 포함하는 것이 좋음.
- 단, 객체가 크거나 상태가 복잡하다면 요약 정보를 담자.
  - ex) "전화번호부(총 12345678개)" 또는 "NetworkGraph(노드 %d개, 엣지 %d개, 밀도: %.2f%%)"
```java
// 그래프 구조
public class NetworkGraph {
    private Set<Node> nodes;
    private Set<Edge> edges;
    
    @Override
    public String toString() {
        return String.format("NetworkGraph(노드 %d개, 엣지 %d개, 밀도: %.2f%%)", 
            nodes.size(), edges.size(), calculateDensity() * 100);
    }
}
```
### 반환 값의 포맷 문서화 여부 결정
- 포맷을 문서화하면 명확성과 일관성을 제공
- 포맷을 문서화하면 그 값을 그대로 입출력에 사용하거나 
  <br> CSV 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장할 수도 있음.
- 문서화할 경우, 해당 포맷과 객체 간 변환 메서드(정적 팩터리 또는 생성자)를 함께 제공하면 좋음.
- 단, 포맷을 한 번 명시하면 향후 변경이 어려움.
  - ex) 포맷에 맞춰 코딩하거나 영구 저장을 한 경우 -> 다시 수정하기 쉽지 않음.
### 포맷 명시 여부와 상관없이 정보를 얻어오는 API 제공를 제공하자
- toString이 반환한 정보를 얻을 수 있는 별도 API를 만들 것.
- toString 반환값을 파싱하게 하지 말 것. -> 성능이 나빠지고 불필요한 작업

### 재정의 할 필요 없는 경우
- 정적 유틸리티 클래스
- 열거 타입(enum) -> 자바에서 이미 완벽한 toString을 제공함.
  - 단, 하위 클래스들이 공유해야 하는 문자열 표현이 있는 추상 클래스라면 재정의 필요.
- 상위 클래스에서 이미 적절히 재정의한 경우

### 정리
- 모든 구체 클래스에서 Object의 toString을 재정의 할 것.
- toString을 재정의한 클래스는 디버깅하기 쉽다!
- 단, 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환 필요.

### 느낀점
- 코딩 테스트 할 때에도, 시간을 사용해서 디버깅하기 편하게 toString을 재정의하면 유용할 것 같습니다.


## 부록
### System.out.println(객체)를 하면?
- 컴파일러가 객체만 출력할 경우 자동으로 toString()을 붙여주고 컴파일을 함!
### 정적 유틸리티 클래스에서 toString을 재정의 할 필요 없는 이유
- 정적 유틸리티 클래스는 인스턴스화 되지 않도록 설게됨. 생성자가 private으로 선언되어 있고, 모든 메서드와 필드가 static
- 정적 유틸리티 클래스는 객체 상태(인스턴스 상태)를 가지지 않음.
- 유틸리티 클래스의 메서드는 일반적으로 클래스 명으로 직접 호출(Collections.sort())하기 때문에 객체 참조 변수로 사용될 일이 거의 없음.
### 정적 유틸리티 클래스
- 특징
  1. 인스턴스 필드를 가지지 않는다. 즉, 상태를 가지면 안된다.
  2. 모든 메서드들은 static으로 되어있다.
  3. 인스턴스화가 될 필요가 없어 이를 방지하기 위해 기본 생성자를 private으로 둔다.
- Java 표준 라이브러리 -> Math
```java
public final class StringUtils {
    private StringUtils() {
        throw new AssertionError("인스턴스화 할 수 없습니다");
    }
    
    public static boolean isEmpty(String str) {
        return str == null || str.trim().length() == 0;
    }
    
    public static String reverse(String str) {
        if (str == null) return null;
        return new StringBuilder(str).reverse().toString();
    }
    
    public static String toTitleCase(String str) {
        if (isEmpty(str)) return str;
        
        StringBuilder builder = new StringBuilder();
        boolean nextTitleCase = true;
        
        for (char c : str.toCharArray()) {
            if (Character.isSpaceChar(c)) {
                nextTitleCase = true;
            } else if (nextTitleCase) {
                c = Character.toTitleCase(c);
                nextTitleCase = false;
            } else {
                c = Character.toLowerCase(c);
            }
            builder.append(c);
        }
        
        return builder.toString();
    }
}
```
### AutoValue 프레임워크
- Google에서 개발한 자바 라이브러리로 불변 값 클래스의 작성을 간소화함.
- 어노테이션을 이용해서 equals(), hashCode(), toString() 등의 메서드를 자동으로 생성
- 단, 각 필드 내용을 멋지게 나타내주지만 정확한 의미까지 파악할 수는 없음.
- 자동 생성된 toString() 예시
  - PhoneNumber{areaCode=707, prefix=867, lineNum=5309}
