# 스트림 병렬화는 주의해서 적용하라

## 자바의 동시성 프로그래밍

- 스레드, 동기화, wait/notify
- java.util.concurrent, Executor 프레임워크
- 포크-조인 (fork-join) 패키지 : 고성능 병렬 분해 프레임워크
- 스트림 : parallel 메서드 호출로 파이프라인 병렬 실행
- 자바에서 동시성 프로그램을 작성하기 점점 쉬워지지만 올바르게 작성하는 것이 중요하다.
- 동시성 프로그래밍과 병렬 스트림 파이프라인 프로그래밍에서 고려해야 할 점 -> `안정성`, `응답 가능 상태 유지`

## 잘못된 사용 예시

- 메르센 소수를 parallel()을 통해 처리할 경우?
  - CPU를 많이 잡아먹으면서 결과는 나오지 않는 상황이 발생할 수 있음
  - 스트림 라이브러리가 파이프라인을 병렬화하는 방법을 찾아내지 못할 수 있음
- 병렬화로 인해 성능 저하가 생길 수 있는 경우
  - 스트림 소스가 Stream.iterate인 경우
  - 중간 연산으로 limit()을 사용하는 경우
- 스트림 파이프라인을 마구잡이로 병렬화하면 성능이 오히려 나빠질 수 있다.

## 가장 병렬화 효과가 좋은 자료구조

- 스트림의 소스가 `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap`의 인스턴스
- 배열 (int[], long[] 등 기본 타입 배열)
- 정수 범위 (IntStream.range, LongStream.range)

```java
//소수인지 확인
import java.util.stream.IntStream;

public class GoodParallelExample {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        long count = IntStream.range(1, 10_000_000)
                .parallel()
                .filter(GoodParallelExample::isPrime)
                .count();

        long end = System.currentTimeMillis();
        System.out.println("Prime count: " + count);
        System.out.println("Time: " + (end - start) + "ms");
    }

    private static boolean isPrime(int n) {
        if (n < 2) return false;
        for (int i = 2; i * i <= n; i++) {
            if (n % i == 0) return false;
        }
        return true;
    }
}
```
- 1000만개 테스트 결과
![image](https://github.com/user-attachments/assets/50dbd30a-9e3a-4132-a7ae-fde428f2b5b7)
- 1억개 테스트 결과


## 위 자료구조들의 특징
 - 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 일을 다수의 스레드에 분배하기 좋음
  - Spliterator를 사용해서 나눌 수 있음 (Stream이나 Iterable의 spliterator 메서드 호출)
- 원소들을 순차적으로 실행할 때 참조 지역성이 뛰어나다!
  - 이웃한 원소의 참조가 메모리에 연속해서 저장되어 있음
   - 참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리에 오기까지 기다려야 함
  - 다양한 데이터를 처리하는 벌크 연산을 병렬화할 때 중요
  - 기본 타입 배열이 가장 참조 지역성이 뛰어남

## 종단 연산 동작 방식

- 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지함
- 순차적인 연산이라면 병렬 수행의 효과가 제한됨
- 병렬화에 가장 적합한 것 : 축소
  - Stream의 reduce, min, max, count, sum 같이 완성된 형태로 제공되는 메서드
  - anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드
  - 반면, 가변 축소를 수행하는 Stream의 collect 메서드는 적합하지 않음

## 올바른 병렬화

- spliterator 메서드를 반드시 재정의하고 결과 스트림의 병렬화 성능을 강도 높게 테스트하라!
- 스트림을 잘못 병렬화하면 성능이 나빠지고 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있음 -> 안전 실패
- 스트림 병렬화는 성능 최적화의 수단 -> 전후 성능 테스트를 통해 가치가 있는지 확인해야 함
- 조건이 잘 갖춰지면 parallel 메서드만으로 프로세서 코어 수에 비례하는 성능 향상을 누릴 수 있음


