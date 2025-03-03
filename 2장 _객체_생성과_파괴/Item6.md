# 불필요한 객체 생성을 피해라
불필요한 객체가 생성되는 상황들을 알려주고 해결 방법을 알려주는 아이템

### String 예시
- 실행될 때마다 새로운 String 인스턴스 생성
  ```
  String s = new String("hi");
  ```
- 하나의 String 인스턴스 사용 <br>
  같은 JVM 내에서 해당 문자열 리터럴을 사용하는 모든 코드가 같은 객체 재사용 <br>
  해당 문자열 리터럴은 *String Pool에 저장됨
  ```
  String s = "hi";
  ```

### static factory 메서드 사용
- 생성자 대신 static factory 메서드를 제공하는 불변 클래스에서는 불필요한 객체 생성을 피할 수 있음
  - 생성자 : 호출할 때마다 새로운 객체 생성
  - static factory : 불변 객체뿐만 아니라 가변 객체도 사용중에 변경되지 않을 것을 안다면 재사용
- ex) Boolean(String) 생성자 대신 Boolean.valueOf(String) factory 메서드 사용
  - Boolean(String) : 매번 새로운 Boolean 객체 생성 ("true" 가 아니라면 모두 false)
  - Boolean.valueOf(String) : 캐싱된 객체 재사용 (Boolean.TRUE or Boolean.FALSE) 
    ```
    public static Boolean valueOf(boolean b) {
      return b ? Boolean.TRUE : Boolean.FALSE;
    }
    ```

### 생성 비용이 비싼 객체는 캐싱해서 사용
- 생성 비용이 비싼 객체가 반복해서 필요할 경우 캐싱해서 재사용
- ex) String.matches : 문자열 형태를 확인하는 쉬운 방법
  - 이 메서드 내부에서 생성되는 Pattern 인스턴스는 1번 사용 후 버려져 GC 대상이 됨
  - Pattern은 입력받은 정규표현식에 해당하는 *유한 상태 머신을 만들기 때문에 인스턴스 생성 비용도 높음 <br>
    근데 이걸 1번 쓰고 버리고 다시 만드니까 더 문제가 됨
  - Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고, 해당 정규표현식을 사용하는 메서드 호출 시마다 재사용
  ```
  public class RomanNumerals {
    //compile : 정규표현식을 컴파일하여 Pattern 객체를 생성하고 정규표현식의 유효성도 검사
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
  }
  ```

### 뒷단 객체 하나당 어댑터 1개씩 생성
- 제 2의 인터페이스 역할 (실제 작업은 뒷단 객체에 위임 -> 뒷단 객체만 관리하면 됨)
- 같은 뒷단 객체를 대변하는 여러 개의 어댑터는 불필요함
  - 여러 개 생성 시 불필요한 객체 증가
  - 뒷단 객체 변경 시 모든 어댑터를 갱신해야 함
  - 일관성 유지 어려움
- ex) Map의 KeySet 메서드 : Map의 모든 Key를 담은 Set 뷰 반환
  - 동일한 Map에서 호출하는 KeySet 메서드는 해당 Map을 대변하는 동일한 뷰 객체 반환
  - 반환된 KeySet 객체 중 하나를 수정하면 다른 KeySet 객체에 동일한 변경 사항이 반영 (동일한 뷰를 공유하기 때문)

### 의도치 않은 오토박싱 주의
- 기본 타입과 박싱된 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술
- 박싱된 기본 타입보다 기본 타입 사용, 의도치 않은 오토박싱이 숨어들지 않도록 주의 (성능 차이↑)
- ex) long 타입인 i가 Long 타입인 sum에 더해질 때마다 불필요한 Long 인스턴스 생성됨
  ```
  private static long sum() {
      Long sum = 0L; // Wrapper 클래스 사용이 문제
      for (long i = 0; i <= Integer.MAX_VALUE; i++)
          sum += i;
      return sum;
  }
  ```
 
### 결론
- "객체 생성은 비싸니 피해야 한다"로 오해하지 말기 <br>
  프로그램의 명확성, 간결성, 기능을 위해 객체 추가 생성은 좋은 일
- 단순 객체 생성을 피하고자 자신만의 *객체 Pool을 만들지 말자
- 이번 아이템은 방어적 복사(아이템 50)와 대조적인 내용
  - 방어적 복사가 필요한 상황에서 객체를 사용했을 때 피해 : 언제 발생할지 모르는 버그, 보안 구멍
  - 필요 없는 객체를 반복 생성했을 때 피해 : 코드 형태, 성능에만 영향 <br>
  ⇒ 후에 아이템 50을 공부할 때 방어적 복사를 막는 게 더 중요하다는 사실을 잊지 말자
  
<hr>

### 부록
- String Pool <br>
  String 리터럴 생성 시 해당 값은 Heap 영역 내 String Constant Pool에 저장되어 재사용 <br>
  String Constant Pool : 문자열 리터럴을 저장하는 영역
- 유한 상태 머신 (Finite State Machine, FSM) <br>
  유한한 개수의 상태(states)와 상태 간의 전이(transitions)로 이루어진 모델 <br>
  특정 입력을 받을 때 미리 정의된 규칙에 따라 상태가 변화하는 구조를 가짐 <br>
  모든 가능한 상태와 전이를 미리 정의하는 과정이 필요해서 생성 비용이 높음
- 객체 Pool : 여러 개의 객체를 미리 만들어 놓고 필요할 때마다 그 객체를 재사용하는 방식
