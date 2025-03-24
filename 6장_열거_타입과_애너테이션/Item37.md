# ordinal 인덱싱 대신 EnumMap을 사용하라
## ordinal을 배열 인덱스로 사용 할 때 문제점
- 배열은 제네릭과 호환되지 않아 비검사 형변환이 필요하고 컴파일 경고 발생
- 인덱스의 의미를 알지 못하므로 출력 시 수동으로 레이블 필요
- 정확한 정수 값을 사용해야 하는 책임이 프로그래머에게 있음
- 타입 안전성이 보장되지 않아 런타임 오류 발생 가능성이 있음
## EnumMap을 이용해서 해결하자
- EnumMap은 열거 타입을 키로 사용하는 고성능 Map 구현체
- 더 간결하고 명확하며 안전한 코드 작성 가능
- 안전하지 않은 형변환이 필요 없음
- 열거 타입 자체가 출력용 문자열을 제공하므로 별도 레이블링 불필요
- 내부적으로 배열을 사용하여 배열의 성능과 Map의 타입 안전성을 모두 제공
## 스트림을 이용한 EnumMap
- 스트림을 사용하면 더 간결하게 작성 가능
- Collectors.groupingBy와 함께 사용할 때 맵 구현체 지정 가능
```java
 System.out.prinfLn(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(LifeCycle.class), toSet())));
```
- EnumMap을 명시적으로 지정하면 공간과 성능 이점 유지
## 중첩으로 EnumMap 이용
- 다차원 관계에서도 중첩된 EnumMap 사용 가능
- Map\<Enum, Map\<Enum,Value>> 형태로 구현
- 새로운 상수 추가 시 EnumMap 버전이 더 안전하고 유지보수가 쉬움
- 내부적으로 맵들의 맵이 배열들의 배열로 구현되어 효율적

## 정리
- 배열 인덱스를 얻기에 ordinal()보단 EnumMap을 사용하자
- 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하자
