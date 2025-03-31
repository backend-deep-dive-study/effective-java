# 반환 타입으로는 스트림보다 컬렉션이 낫다
## 기존 자바 7까지의 원소 시퀀스 반환 방식
- 원소 시퀀스를 반환할 때 주로 Collection, Set, List, Iterable, 배열을 사용
  - 기본 : Collection 인터페이스
  - for-each문 또는 반환된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을 때 : Iterable
  - 기본 타입 또는 성능이 중요 : 배열
## 자바 8 이후 스트림 등장
### 스트림의 반복 미지원
- 스트림은 반복(iteration)을 직접 지원하지 않음
- 스트림만 반환하면, for-each 같은 반복문을 사용하려는 사용자는 불편함.
- Stream은 Iterable처럼 보이지만, Iterable을 상속하지 않아 for-each에서 직접 사용이 불가!
```java
// 컴파일 오류 발생
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator){  
}
```
### 해결 방법 - 어댑터 메서드 이용
```java
// 비추천 방식 (난잡하고 직관성이 떨어짐!)
for (ProcessHandle ph : (Iterable<ProcessHandle>)
 ProcessHandle.allProcesses()::iterator){
}

// Stream을 Iterable로 변환(형 변환 불필요!)
// 반복문에서만 쓰이는 경우
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

// Iterable을 Stream으로 변환
// Stream 파이프 라인에서만 쓰이는 경우
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```
## 공개 API를 만들 때
- 스트림과 반복문에서 쓰려는 사용자를 모두 고려해야함.
- Collection이나 그 하위 타입을 쓰는게 최선
  - Collection은 Iterable이기도 하며 stream()도 지원하므로 양쪽 모두 사용 가능!
## Collection 반환 예외 상황
- 대용량 시퀀스나 무한 시퀀스는 Collection으로 반환하면 위험
  - 모든 원소를 메모리에 적재하게 됨
### 대안 방법
- 원소 개수가 많지만 규칙성이 있는 경우
  - 전용 컬렉션 구현을 고려
  - ex) AbstractList를 상속해 멱집합(부분집합) 구현
    - 최소 구현 메서드
    - size(), contains(Object), Iterator를 제공하는 iterator() 메서드
  - ex) 부분리스트
    - Stream을 이용해 리스트의 모든 부분리스트 반환
    ```java
    public class SubLists {
        public static <E> Stream<List<E>> of(List<E> list) {
            // 빈 리스트와 모든 부분리스트를 연결
            return Stream.concat(Stream.of(Collections.emptyList()), 
                prefixes(list).flatMap(SubLists::suffixes));
        }
        
        // 리스트의 모든 프리픽스(접두사)를 생성
        // 예: [a, b, c]의 프리픽스는 [a], [a, b], [a, b, c]
        private static <E> Stream<List<E>> prefixes(List<E> list) {
            return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
        }

        // 리스트의 모든 서픽스(접미사)를 생성
        // 예: [a, b, c]의 서픽스는 [a, b, c], [b, c], [c]
        private static <E> Stream<List<E>> suffixes(List<E> list) {
            return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
        }
    }
    ``` 
## 성능 및 코드 품질
- 어댑터 방식은 코드가 어수선해지고 성능도 느릴 수 있음!
- 전용 컬렉션은 코드가 복잡하지만 성능은 상대적으로 좋음!
## 정리
- 원소 시퀀스를 반환할 때는 스트림과 반복을 모두 고려하자.
- 원소 개수가 적다면 Collection을 반환
- 불가능하다면 전용 컬렉션을 고민해보자.
- 컬렉션을 반환하는게 불가능한 경우
  - Stream과 Iterable 중 더 자연스러운 방식을 사용!
## 부록
### 원소 시퀀스란?
- 일련의 데이터 항목들을 순서대로 나열한 것. 자바에서는 보통 컬렉션, 배열, 스트림 등으로 표현
