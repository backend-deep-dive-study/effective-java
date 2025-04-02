# 다중정의는 신중히 사용하라
## 다중정의와 재정의
- 다중정의(Overloading): 컴파일타임에 호출할 메서드가 결정됨 (매개변수의 정적 타입 기준)
- 재정의(Overriding): 런타임에 호출할 메서드가 결정됨 (객체의 실제 타입에 따라 결정)
### 예시
- 컬렉션 분류기
```java
public class CollectionClassifier {
    public static String classify(Set<?> s) { 
        return "집합";
    }
    
    public static String classify(List<?> lst) { 
        return "리스트";
    }
    
    public static String classify(Collection<?> c) { 
        return "그 외";
    }
    
    public static void main(String[] args) {
        Collection<?>[] collections = { 
            new HashSet<String>(), 
            new ArrayList<BigInteger>(), 
            new HashMap<String, String>().values() 
        };
        // 컴파일 타임에 for문 안의 c는 항상 Collection<?> 타입!
        for (Collection<?> c : collections) 
            System.out.println(classify(c)); 
    }

    // 해결법 (instanceof를 사용)
    public static String classify(Collection<?> c) {
        return c instanceof Set ? "집합" :
            c instanceof List ? "리스트" : "그 외";
        }
    }
```
- 재정의된 메서드 호출
```java
class Wine {
    String name() { return "포도주"; } 
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) { 
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champagne());
        for (Wine wine : wineList)
            System.out.println(wine.name());
    } 
}
```
- 오토박싱 혼란
  - remove(int)와 remove(Object)는 다중정의되어 있고, <br>
    int는 오토박싱되어도 여전히 int 우선 → 인덱스로 처리됨 <br>
    결과적으로 의도치 않은 요소 제거 발생
```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();
        
        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);  // 인텔리제이 ide에서는 모호하다고 알려줌
        }
        
        System.out.println(set + " " + list);

        // 해결법
        list.remove((Integer) i); // 명시적 형변환
    }
}
```
- 메서드 참조와 람다
  - submit(Runnable)과 submit(Callable<T>)이 다중정의되어 있어 혼란<br>
    System.out::println이 Runnable인지 Callable인지 애매 → 컴파일러가 구분 X<br>
    System.out::println이 오버로드된 메서드라 더욱 복잡
```java
// Thread의 생성자 호출 - 정상 동작
new Thread(System.out::println).start();

// ExecutorService의 submit 메서드 호출 - 컴파일 오류
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);  // 컴파일 오류
```

## 다중정의할 때 지킬 것
1. 매개변수 수가 같은 다중정의는 가능한 피할 것
2. 가변인수(varargs)를 사용하는 메서드는 다중정의하지 말 것
3. 매개변수가 근본적으로 다른 경우(서로 형변환 불가능한 타입)에만 다중정의 고려
4. 다중정의보다 메서드 이름을 다르게 짓는 방법을 고려할 것 (예: ObjectOutputStream의 writeBoolean, writeInt 등)
5. 생성자는 이름을 다르게 지을 수 없으니 정적 팩터리 메서드 활용 고려
6. 람다와 메서드 참조를 사용할 때 주의 필요 (특히 함수형 인터페이스를 매개변수로 받는 경우)
7. 다중정의된 메서드들이 동일 입력에 대해 동일한 동작을 하도록 구현하면 혼란 최소화 가능
   ex) 인수 포워드
    ```java
    public boolean contentEquals(StringBuffer sb) {
        return contentEquals((CharSequence) sb);
    }
    ```
## 정리
- 프로그래밍 언어에서 다중정의를 지원한다고 해서 꼭 활용할 필요는 없으며, 특히 매개변수 수가 같을 때는 피하는 것이 좋다. 
- 불가피한 경우 형변환을 통해 정확한 메서드가 호출되도록 하거나, 같은 입력에 대해 동일하게 동작하도록 구현하자!

## 부록
### Callable이란?
- Java의 동시성 API에서 사용되는 함수형 인터페이스
- Runnable과의 차이점
  - 반환 값:
    - Runnable의 run() 메서드는 반환 값이 없음(void)
    - Callable의 call() 메서드는 값을 반환
  - 예외 처리:
    - Runnable은 체크드 예외를 던질 수 없음
    - Callable은 체크드 예외를 던질 수 있음
