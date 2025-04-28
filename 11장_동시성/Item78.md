# 공유 중인 가변 데이터는 동기화해 사용하라

### 동기화가 필요한 이유
- 멀티스레드 환경에서는 스레드들이 공유하는 가변 데이터를 동시에 읽고 쓸 수 있음
- 이때 컴파일러 최적화, CPU 캐시 등으로 인해, 수정된 값이 다른 스레드에 보이지 않을 수 있음

### 동기화하지 않은 경우 문제
```java
private static boolean stopRequested;

public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
        while (!stopRequested) {
            // 작업 수행
        }
    });

    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true; // 플래그를 true로 설정
}
```
  - stopRequested 가 true여도 backgroundThread가 무한루프를 빠져나오지 못할 수 있음

### 동기화 방법
1. synchronized
- 읽기/쓰기를 모두 동기화하면 메모리 가시성 문제와 동시 접근 모두 안전

```java
private static boolean stopRequested;

private static synchronized void requestStop() {
    stopRequested = true;
}

private static synchronized boolean stopRequested() {
    return stopRequested;
}

public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
        while (!stopRequested()) {
            // 작업 수행
        }
    });

    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    requestStop(); // 안전하게 중지 요청
}
```
2. volatile
- 값을 읽고 쓸 때 메모리에 즉시 반영
- 다른 스레드가 항상 최신 값을 읽도록 보장
- 복합 연산 (읽고 수정하고 다시 저장하는)의 경우는 synchronized 필요

3. 고수준 동시성 도구 사용
- AtomicBoolean, AtomicInteger

### 동기화의 기능
1. 가시성 확보 : 하나의 스레드가 작성한 변경을 다른 스레드가 볼 수 있도록 함
2. 상호배제 : 동시에 접근하는 문제 해결

 즉, 데이터 일관성 유지를 위해 필요!

### 동기화 없이도 안전한 상황
- final 필드만 존재하는 객체
- 불변 객체
- 스레드 간 공유가 없는 경우 (독립된 인스턴스를 사용하는 경우)

### 결론 : 공유하는 가변 데이터는 반드시 동기화하거나 아니면 공유하지 말자!

## 부록
### synchronized vs volatile
| 항목 | synchronized | volatile |
|:---|:---|:---|
| 목적 | 상호배제(Mutual Exclusion) + 가시성(Visibility) 보장 | 가시성(Visibility)만 보장 |
| 사용하는 상황 | - 여러 스레드가 같은 데이터에 읽기/쓰기 할 때<br>- 복합 연산(읽기-수정-쓰기)이 필요한 경우 | - 단순 읽기/쓰기만 있을 때<br>- 복합 연산이 필요 없는 경우 |
| 작동 방식 | - 하나의 스레드만 임계 구역 접근 허용<br>- 진입/탈출 시 메모리 동기화 수행 | - 변수의 읽기/쓰기를 메모리에서 직접 수행<br>- CPU 캐시 무시, 항상 최신 값 읽음 |
| 성능 비용 | 비교적 크다 (락 획득/해제 비용 존재) | 매우 작다 (락 없이 가시성만 보장) |
| 코드 예시 | ```java<br>synchronized(lock) { count++; }<br>``` | ```java<br>volatile boolean stopRequested;<br>``` |
| 주의할 점 | - 교착 상태(deadlock) 위험 있음<br>- 긴 임계 구역은 성능 저하 가능 | - 복합 연산(예: count++)은 여전히 안전하지 않음 |
| 대표 사용 예시 | - 공유 컬렉션 수정<br>- 중요한 상태 변경 | - 플래그(stopRequested) 설정<br>- 단순 상태 체크 |

- **synchronized**: 읽기-수정-쓰기 복합 작업 필요할 때
- **volatile**: 단순한 읽기/쓰기만 필요할 때

> 상호배제까지 필요하면 synchronized, 읽기/쓰기만 최신화하면 volatile을 사용한다.
