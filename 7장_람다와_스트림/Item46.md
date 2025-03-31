# 스트림에서는 부작용 없는 함수를 사용하라

### 스트림이란?

- 함수형 프로그래밍에 기초한 패러다임
    - 함수형 프로그래밍: 자료 처리를 함수의 계산으로 취급하고 상태와 가변 데이터를 멀리하는 프로그래밍 패러다임
- 스트림이 제공하는 표현력, 속도, 병렬성을 얻기 위해서는 패러다임 이해 필요

### 스트림의 패러다임

- 계산을 일련의 변환으로 재구성
- 각 변환 단계는 ~~가능하면~~ 이전 단계의 결과를 받아 처리하는 순수 함수여야 함
    - 순수함수: 입력만이 결과에 영향을 주는 함수 → 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않음
- 스트림 연산에 들어가는 함수 객체는 모두 부작용이 없어야함

### 단어 빈도표 예제

#### 부적절한 사용

```java
// 파일의 단어를 소문자로 변경 후 단어별 수를 세어 빈도표를 만드는 코드
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
	});
}
```

- 모든 데이터 처리 작업이 *최종 연산인 forEach에서 발생
- 외부 상태(freq)를 수정하는 람다를 실행하며 문제 발생
- forEach가 스트림이 수행한 연산 결과를 보여주는 일 이상 처리
    - forEach는 스트림 계산 결과를 보고할 때만 사용하는 것을 권장
    - 계산할 때는 사용하지마!

#### 적절한 사용

```java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
	freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

- Map 객체를 새로 생성하여 할당 → 수정 발생X

### Collector, 수집기

- 다음 예시들에서는 축소 전략을 캡슐화한 블랙박스 객체라 생각
    - 축소: 스트림의 원소들을 객체 하나에 취합
- 수집기 사용 시 스트림의 원소를 쉽게 컬렉션으로 모을 수 있음
- 수집기 종류: `toList()` , `toSet()` , `toCollection(collectionFactory)` 등

### toList()

```java
List<String> topTen = freq.keySet().stream()
            .sorted(comparing(freq::get).reversed())
            .limit(10)
            .collect(toList());
```

- comparing: 키 추출 함수를 받는 비교자 생성 메서드
    - freq::get은 입력받는 단어를 빈도표에서 찾아 그 빈도를 반환
- 가장 흔한 단어가 위로 오도록 reverse sort

### toMap()

#### 기본 형태

```java
Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

- 간단한 맵 수집기 `toMap(keyMapper, valueMapper)`
- 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받음
- 각 원소가 고유한 키에 매핑되어 있을 때 적합
- 스트림 원소 다수가 같은 키를 사용하면 파이프라인이 IllegalStateException을 던지며 종료

#### 충돌 처리 버전

- merge 함수 제공
- 동일한 키에 대해 충돌이 발생할 경우 처리
    - 같은 키를 공유하는 값들은 merge 함수를 사용해 기존 값에 합쳐짐
    - 마지막 값 선택 방식 사용 가능 (last-write-wins)
        
        ```java
        toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
        ```
        

### groupingBy()

```java
// 알바벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 
words.collect(groupingBy(word -> alphabetize(word)))
```

- 입력으로 분류 함수를 받고 출력으로 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기 반환
- 이 카테고리는 해당 원소의 키로 사용

#### 다운스트림 수집기 예시

- 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면 분류 함수와 함께 다운스트림 수집기도 명시
    - 다운스트림: 그룹핑 계열 수집기 내부에서 각 그룹의 값을 어떻게 처리할지를 정하는 수집기
- 다운스트림 수집기는 해당 카테고리(맵의 키)의 모든 원소를 담은 스트림으로부터 값을 생성
    - `toSet()` 사용 시 원소들의 집합을 값으로 갖는 맵 생성
    - `toCollection(collectionFactory)` 사용 시 컬렉션을 값으로 갖는 맵 생성
    - `counting()` 사용 시 각 카테고리(키)를 해당 카테고리에 속하는 원소의 개수와 매핑한 맵 생성
        
        ```java
        // 소문자로 변환한 각 단어의 등장 횟수를 계산하는 맵 생성
        Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
        ```
        
- `groupingByConcurrent` : 동시성 지원 → ConcurrentHashMap 인스턴스 생성
- `partitioningBy` : 분류 함수 자리에 프레디키드(입력값을 받아 true or false를 반환)를 받고 키가 Boolean인 맵을 반

### joining()

- 문자열 등의 CharSequece 인스턴스의 스트림에만 적용 가능
- 매개변수가 없는 `joining` 은 단순 원소들을 연결하는 수집기 반환
- 매개변수가 1개인 경우는 CharSequece 타입의 구분문자를 받음
    - 연결 부위에 구분문자 삽입
- 매개변수가 3개인 경우는 접두, 구분, 접미문자를 받음

### 정리

- 스트림 파이프라인 프로그래밍 핵심은 부작용 없는 함수 객체
- 스트림 관련 객체에 전달되는 모든 함수 객체가 부작용이 없어야함
- forEach(종단 연산)은 계산 결과 보고 시에만 사용 (계산 자체에 이용X)
- 스트림을 사용하려면 수집기를 잘 알아둬야함
    - toList, toSet, toMap, groupingBy, joining

---

## 부록

### 최종 연산 (종단 연산)

- 스트림을 소모하면서 결과를 생성함
- 예: `forEach`, `collect`, `count`, `reduce`, `anyMatch` 등
- 종단연산을 호출하면 **스트림은 더 이상 사용할 수 없음**

```java
stream.forEach(System.out::println);  // 종단연산
```

### 중간 연산

- 스트림을 변환하거나 필터링하는 연산
- 지연 실행: **종단연산이 호출되기 전까지 실제로 실행되지 않음**
- 스트림을 반환하기 때문에 연쇄적으로 연결할 수 있음

```java
Stream<String> stream = Stream.of("a", "b", "c")
    .map(String::toUpperCase)  // 중간연산
    .filter(s -> !s.equals("B"));  // 중간연산
```

### 전체 흐름

```java
List<String> data = List.of("apple", "banana", "apricot");

long count = data.stream()               // 스트림 생성
    .filter(s -> s.startsWith("a"))      // 중간연산
    .map(String::toUpperCase)            // 중간연산
    .count();                            // 종단연산
```
