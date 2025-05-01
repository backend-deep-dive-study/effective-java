# wait와 notify보다는 동시성 유틸리티를 애용하라

### wait과 notify

- 고수준 동시성 유틸리티가 wait과 notify로 하드코해야 했던 일들을 대신 처리
- wait과 notify는 올바르게 사용하기 까다로움 → 고수준 동시성 유틸리티 사용하자

### java.util.concurrent의 고수준 유틸리티

- 실행자 프레임워크
- 동시성 컬렉션
- 동기화 장치

### 동시성 컬렉션

- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션
- 높은 동시성을 위해 동기화를 각자 내부에서 수행
    - 동시성 컬렉션에서 동시성 무력화 불가능
    - 외부에서 락을 추가로 사용하면 속도 저하

### 상태 의존적 메서드

- 동시성 무력화가 불가능하여 여러 메서드를 원자적으로 묶어 호출하는 일도 불가능
- 여러 기본 동작을 하나의 원자적 동작으로 묶는 ‘상태 의존적 수정’ 메서드들이 추가됨
- ex) Map의 putIfAbsent(key, value)
    - 주어진 키에 매핑된 값이 없을 때만 새 값을 넣음
    - 기존 값이 있었다면 그 값을 반환, 없으면 null 반환
    - String.intern의 동작 구현해보기 (JVM 상수 풀이 아닌 ConcurrentHashMap에 저장)
    
    ```java
    private static final ConcurrentMap<String, String> map = 
    		new ConcurrentHashMap<>();
    
    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null) {
                result = s;
            }
        }
        return result;
    }
    ```
    
    - 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만듦
        - ex) Collections.synchronizedMap보다 ConcurrentHashMap 사용이 더 좋음
        - 동기화된 맵을 동시성 맵으로 교체하는 것만으로 동시성 애플리케이션의 성능 개선 가능

### 작업 성공 대기

- 컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 대기하도록 확장됨
- ex) Queue를 확장한 BlockingQueue
    - take: 큐의 첫 원소를 꺼냄 → 큐가 비면 새로운 원소가 추가될 때까지 기다림
    - BlockingQueue는 작업 큐(생산자-소비자 큐)로 쓰기 적합

### 동기화 장치

- 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해줌
- 자주 사용: CountDownLatch, Semaphore
- 덜 사용: CyclicBarrier, Exchanger
- 가장 강력: Phaser

### CountDownLatch

- 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 함
- CountDownLatch의 유일한 생성자는 int 값을 받으며, 이 값이 Latch의 countDown 메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지 결정
- ex) 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 프레임워크 구축
    - wait, notify로 구현 가능 → CountDownLatch로 간단&직관적 구현
    
    ```java
    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);
    
        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }
    
        ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
    ```
    
    - countDownLatch 3개 사용
        - ready: 작업자 스레드들이 준비가 됐음을 타이머 스레드에 통지
            - 통지를 끝낸 작업자 스레드들은 start가 열리길 기다림
            - 마지막 작업자 스레드가 ready.countDown을 호출 → 타이머 스레드가 시작 시각 기록 + start.countDown 호출
        - start: 기다리던 작업자 스레드들을 깨움
            - 타이머 스레드는 done이 열리길 기다림
            - 마지막 작업자 스레드가 동작 종료 + done.countDown 호출
        - done: 해당 Latch가 열리면 종료 시각 기록
    - time 메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 함
        - 이렇게 설계하지 않으면 메서드가 끝나지 않음 → 스레드 기아 교착상태 발생
    - 시간을 잴 때는 시스템 시간과 무관한 System.nanoTime을 사용하는 것이 더 정확

### wait과 notify 사용

- notify: wait 중인 스레드 하나 깨움
- wait: 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용
    - lock 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 함
    
    ```java
    synchronized (obj) {
        while (조건이 충족되지 않았다) {
            obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
        }
        ... // 조건이 충족됐을 때의 동작을 수행한다.
    }
    ```
    
    - wait 사용 시 반드시 대기 반복문 사용 → 밖에서 호출 금지
    - 대기 전에 조건을 검사하여 조건 충족 시 wait을 건너뛰어 응답 불가 상태 예방
    - 이미 조건이 충족됐는데 스레드가 notify 메서드를 먼저 호출한 후 대기 상태로 빠지면 스레드를 다시 깨울 수 없을지도..
- 대기 후 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치
    - 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불벽식 깨질 수 있음
- 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황
    - 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드 락을 얻어 그 락이 보호하는 상태를 변경
    - 조건이 만족되지 않았지만 다른 스레드가 실수/악의적으로 notify를 호출하는 경우
    - 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨우는 경우
    - 대기중인 스레드가 드물게 notify 없이도 깨어나는 경우 존재 → 허위 각성 현상

### notify vs notifyAll

- notifyAll 사용 권장: 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것
    - 다른 스레드도 깨어나지만 해당 프로그램의 정확성에 영향X
    - 기다리던 조건이 충족됐는지 확인 후 충족되지 않았으면 다시 대기하기 때문
- 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify로 최적화 가능
- 그래도 notifyAll 권장
    - notify 대신 notifyAll 사용 시 관련 없는 스레드가 실수/악의적으로 wait을 호출하는 공격으로부터 보호 가능
    - ~~외부로 공개된 객체에 대해 실수/악의적으로 notify를 호출하는 상황에 대비하기 위해 wait울 반복문 안에서 호출한 것처럼~~

### 정리

- 새로 작성하는 코드라면 동시성 유틸리티 사용하자
- wait과 notify를 사용해야하면
    - wait은 항상 while문 안에서 호출
    - notify 대신 notifyAll 사용
    - notify 사용 시 응답 불가 상태에 빠지지 않도록 주의

---

## 부록

### String.intern

1. 해당 문자열이 상수 풀에 존재하는지 확인
2. 존재하면, 상수 풀에 있는 문자열의 참조 반환
3. 존재하지 않으면, 현재 문자열을 상수 풀에 추가하고 그 참조 반환
