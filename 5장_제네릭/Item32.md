# 제네릭과 가변인수를 함께 쓸 때는 신중하라
## 문제점
- 가변인수 메서드 호출 시 배열이 자동으로 생성
  - 배열이 클라이언트에게 노출될 수 있어 위험
- 제네릭과 매개변수화 타입은 실체화되지 않음
  - 제네릭 타입은 런타임에 타입 정보를 유지하지 않음
  - 가변인수와 함께 사용하면 **힙 오염**이 발생할 수 있음
- 제네릭 타입의 가변인수를 사용할 경우, 컴파일러는 "unchecked"경고를 발생
### 잘못된 예시
```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList;  // 힙 오염 발생
    String s = stringLists[0].get(0); // ClassCastException 발생
}
```
- stringLists는 List\<String>[] 타입의 배열이지만, Object[]로 변환 가능
- Object[]에 List\<Integer>를 할당하면 힙 오염이 발생하여, 실행 시 ClassCastException이 발생
### 제네릭 배열은 금지하면서 제네릭 가변인수는 허용하는 이유
- 실무에서 제네릭 varargs 메서드가 매우 유용하기 때문
- 자바 라이브러리에서 타입 안전한 제네릭 varargs 메서드들
  - Arrays.asList(T... a)
  - Collections.addAll(Collection<? super T> c, T... elements)
  - EnumSet.of(E first, E... rest)

## 제네릭 메서드가 안전한 조건
1. vargargs 매개변수 배열에 아무것도 저장하지 않음
2. 그 배열(또는 복제본)을 신뢰할 수 없는 코드에 노출하지 않음

## 제네릭 가변인수를 안전하게 사용하는 방법
1. @SafeVarargs 애너테이션 사용
   - 제네릭 가변인수 메서드의 안전성이 보장될 때 사용
   - 자바 8이상부터 static, final, private 메서드에만 적용 가능 
2. List로 대체
   - 제네릭 varargs를 List 매개변수로 대체
   - List.of() 같은 정적 팩토리 메서드 활용
   - 컴파일러가 타입 안전성 검증 가능
## 정리
- 가변인수와 제네릭 궁합은 좋지 않음.
  - 배열을 노출하여 추상화가 완벽하지 못함
  - 배열과 제네릭 타입 규칙이 서로 다름
- 제네릭 varargs 매개변수는 타입 안전하지 않지만 실용성을 위해 허용
- 메서드에 제네릭(혹은 매개변수화된) varargs를 사용하려면
  - 메서드가 타입 안전한지 확인
  - 타입 안전한 경우 @SafeVarargs 애너테이션을 달자
## 부록
### 가변인수란
- 개수가 정해지지 않은 인수를 배열 형태로 받을 수 있게 해주는 기능
### 힙 오염이란
- 매개변수화 타입의 변수가 타입이 다른 객체를 참조할 때 발생하는 문제
### varargs란
- 가변인수를 지원하기 위해 사용하는 문법 (T... args)
