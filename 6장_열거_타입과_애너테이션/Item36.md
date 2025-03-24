# 비트 필드 대신 EnumSet을 사용하라

### 비트 필드
- 열거한 값들이 집합으로 사용될 경우, 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴
- 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모은 것
- 예시) 회원 권한 부여
    - 읽기 권한 (Read) → `0001`
    - 쓰기 권한 (Write) → `0010`
    - 실행 권한 (Execute) → `0100`
    - 삭제 권한 (Delete) → `1000`
    ```java
    int permission = 0;
    permission = permission | READ | WRITE; // 읽기, 쓰기 권한 추가 (비트 OR 연산으로 합침) -> 0011
    ```
    

### 비트 필드 문제점
- 비트 필드를 그대로 출력 시 단순 정수 열거 상수보다 해석 어려움
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다로움
- 최대로 필요한 비트 수를 예측하여 적절한 타입 선택 필요
    - API 수정하지 않고 비트 수를 늘릴 수 없음

### EnumSet 사용
- 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현
- Set 인터페이스 완벽히 구현
- 타입 안전
- 다른 어떤 Set 구현체와 함께 사용 가능
- 비트 벡터로 구현됨
    - 원소가 64개 이하일 경우, EnumSet 전체를 long 변수 1개로 표현 → 비트 필드와 비슷한 성능
    - removeAll, retainAII 같은 대량 작업 → 비트를 효율적으로 처리할 수 있는 산술 연산 사용
    - 비트를 직접 다룰 때 발생하는 오류 방지
```java
public class Text {
		public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
		
		public void applyStyles(Set<Style> styles) { ... }
}
```
- Set<Style> 인터페이스로 받기 → 다른 Set 구현체를 넘기더라도 처리 가능

### 정리
- 열거할 수 있는 타입을 모아 집합 형태로 사용해도 비트 필드를 사용할 이유 X
- EnumSet이 비트 필드와 비슷한 성능, 열거 타입의 장점 제공
    - 타입 안전성 : 잘못된 값을 컴파일 타임에 차단
    - 가독성 및 유지보수성
    - 이름 충돌 X
    - 필드, 메서드 추가 가능
    - 직렬화와 싱글턴 특성 보장 : JVM에서 직렬화 자동 처리, 싱글턴 특성 안전하게 유지
- EnumSet의 유일한 단점 → 불변 EnumSet 제작 불가
    - (성능, 명확성 조금 희생) Collections.unmodifiableSet으로 EnumSet 감싸서 사용
