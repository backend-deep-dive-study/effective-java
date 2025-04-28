# 과도한 동기화는 피하라

- 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 예측할 수 없는 동작을 낳기도 한다.

## 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.

- 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다.
  - 동기화된 영역을 포함한 클래스 관점에서는 모두 바깥 세상에서 온 외계인이다.

### 잘못된 코드, 동기화 블록 안에서 외계인 메서드를 호출하는 경우
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }
    
    private final List<SetObserver<E>> observers = new ArrayList<>();
        
    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        // 이 부분이 쟁점
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }
    
    @Override public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }
    
    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // notifyElementAdded가 호출된다.
        return result;
    }
}
```

```java
@FunctionalInterface
public interface SetObserver<E> {
    // ObserwbleSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```

- 아래 코드는 0~99까지 출력하며 정상적으로 동작한다.

    ```java
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver((s, e) -> System.out.println(e));

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
    ```

- 하지만 만약 아래 코드로 작성하게 된다면 문제가 생길 수 있다.

    ```java
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23)
                    s.removeObserver(this);
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
    ```

    - 0~23까지 출력한 후 관찰자 자신을 구독해지한 다음 조용히 종료하길 원하지만, 실제로 실행해보면 `ConcurrentModificationException`이 발생한다.

    - added 메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출한다.

    - notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.

- 이번에는 구독해지를 하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고 실행자 서비스를 사용해 다른 스레드한테 부탁해보자.

    ```java
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23)
                    ExecutorService exec = Executors.newSingleThreadExecutor();
                try {
                    exec.submit(() -> s.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
    ```

    - 이 코드는 예외는 나지 않지만 교착상태에 빠지게 된다.
      - 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도하지만 락을 얻을 수 없다. 메인 스레드가 이미 락을 쥐고 있기 때문이다. 

### 해결책 1. 외계인 메서드를 동기화 블록 바깥으로 옮기기

```java
    private void notifyElementAdded(E element) {
        List<SetObserver<E>> snapshot = null;
        synchronized(observers) {
            snapshot = new ArrayList<>(observers);
        }
        for (SetObserver<E> observer : observers)
            observer.added(this, element);
    }
```

- 이 방식으로 하게 된다면, 앞서의 두 예제에서 예외 발생과 교착상태 증상이 사라진다.

### 해결책 2. 자바의 동시성 컬렉션 라이브러이의 CopyOnWriteArrayList 사용하기

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

- 해당 클래스는 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다.
  - 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다.
  - 수정할 일이 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 최적이다.(다른 용도로 사용지 끔찍이 느릴 수 있다.)

- 동기화 영역에서는 가능한 한 일을 적게 하는 것이 좋다.

## 성능 측면에서의 동기화

- 이제 정확성에 관한 이야기가 아닌 성능 측변에서의 비용을 알아보자.

- 자바의 동기화 비용은 빠르게 낮아져 왔다. 멀티코어가 일반화된 오늘날, 과도한 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아니다.
- 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 **지연시간**이 진짜 비용이다.

- 가변 클래스를 작성하려거든 두 선택지 중 하나를 따르자.

    1. 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
    2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자.<br>
    (클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방식을 선택해야 한다.)

- java.util은 (구식인 Vector, Hashtable을 제외하고) 첫 번째 방식을 취했고, java.util.concurrent는 두 번째 방식을 취했다.

# 결론

교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자.

일반화해 이야기하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자.

가변 클래스를 설계할 때는 스스로 동기화해야 할지 고민하자.

멀티코어 세상인 지금은 과도한 동기화를 피하는 게 과거 어느 때보다 중요하다.

합당한 이유가 있을 때만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자자.
