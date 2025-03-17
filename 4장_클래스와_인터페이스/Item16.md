# public 클래스에서는 public 필드가 아닌 접근자 메서드를 이용해라

### 인스턴스 필드만 모아둔 목적 없는 클래스의 문제점

- 데이터 필드에 직접 접근 가능 (캡슐화 이점x)
- API 수정 없이 내부 표현 변경 불가
- 불변식 보장x
- 외부에서 필드 접근 시 부수 작업 수행 불가

### 필드를 모두 private로 바꾸고 public 접근자(getter) 추가

- 패키지 밖에서 접근할 수 있는 public 클래스의 경우
    - 접근자를 제공하여 클래스 내부 표현 방식을 바꿀 수 있는 유연성 획득
- public 클래스가 필드를 공개하면 해당 필드를 사용하는 클라이언트가 생길 것이므로 내부 표현 방식 변경이 어려움

### package-private 클래스 / private 중첩 클래스 → 데이터 필드 노출에 문제가 없음

- *package-private 클래스
    - 같은 패키지 내에서만 접근 가능
    - public 필드를 만들어도 그 필드를 제어할 수 있는 코드가 패키지 내부에만 존재
    - 필드 노출로 캡슐화 위반 문제가 크지 않음
- private 중첩 클래스
    - outer 클래스 내부에서만 사용되는 클래스
    - outer 클래스가 해당 inner 클래스를 완전히 통제 가능
    - 외부에서 inner 클래스 인스턴스 생성도 불가능하여 접근할 방법x

### public 클래스의 필드를 직접 노출하는 라이브러리

- Point
- Dimension의 성능 문제
    
    ```java
    public class Dimension {
        public int width;
        public int height;
    
        public Dimension(int width, int height) {
            this.width = width;
            this.height = height;
        }
    }
    ```
    
    - 불변성 위반 (객체의 상태 예측 불가)
    - 스레드 안정성 문제
    - 최적화 기회 감소 (필드 변경 가능성을 고려해야 해서 최적화 제한)

### public 필드가 불변일 경우

- (해결) 불변성 위반
- (해결) 불변식 보장
- API 변경 없이 표현 방식 변경 불가
- 필드 접근 시 부수 작업 수행 불가

### 정리

- public 클래스의 가변 필드는 노출하면 안됨
- 불변 필드일 경우 노출해도 위험성 요소가 줄지만 완벽한 방법은 아님
- package-private 클래스, private 중첩 클래스에서는 필드 노출이 좋을 때도 있음
