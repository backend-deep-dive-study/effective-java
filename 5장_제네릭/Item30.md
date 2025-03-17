# 이왕이면 제네릭 메서드로 만들라

> 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다. 예컨대 Collections의 ‘알고리
즘’ 메서드(binarysearch, sort 등)는 모두 제네릭이다.


### 로 타입은 사용하지 말라(아이템 26)

```
public static ____ Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```
> 컴파일은 되지만 경고가 두개 발생한다. 경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다. 메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

> 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.(밑줄부분) 다음 코드에서 타입 매개변수 목록은 `<E>`이고 반환 타입은 `Set<E>`이다. 타입 매개 
변수의 명명 규칙은 제네릭 메서드나 제네릭 타입이나 똑같다.(아이템 29,68)

```
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

- 장점
	- 타입 안전함 보장
	- 유연함
	- 쓰기가 쉬움

### 제네릭 싱글턴 팩터리 패턴

> 때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입 정보가 소거(아이템 28)되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다.

- Collections.emptySet()
> 컴파일 시점: 제네릭 타입(`Set<String>`, `Set<Integer>`)은 다르게 보이지만, 사실 제네릭 타입이 소거(타입 소거, Type Erasure)되기 때문에 실제로는 런타임에 모두 같은 Set 타입으로 처리됩니다. <br>
런타임 시점: 제네릭은 타입 정보를 소거하기 때문에, `Set<String>`, `Set<Integer>` 모두 실제로는 동일한 Set 객체로 취급됩니다.

```
import java.util.HashSet;
import java.util.Set;

public class WrongExample {
    public static <T> Set<T> createEmptySet() {
        return new HashSet<>(); // 새로운 객체를 계속 생성
    }

    public static void main(String[] args) {
        Set<String> stringSet = createEmptySet();
        Set<Integer> intSet = createEmptySet();

        System.out.println(stringSet == intSet); // false (다른 객체)
    }
}
```

```
import java.util.Collections;
import java.util.Set;

public class GenericSingletonFactory {
    // 하나의 빈 Set 객체
    private static final Set<?> EMPTY_SET = Collections.emptySet();

    // 제네릭 메서드로 요청된 타입에 맞게 동일한 Set 객체 반환
    @SuppressWarnings("unchecked")
    public static <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;  // 타입 캐스팅을 통해 반환
    }

    public static void main(String[] args) {
        Set<String> stringSet = emptySet();
        Set<Integer> intSet = emptySet();

        System.out.println(stringSet == intSet); // true (동일 객체)
    }
}
```

### 재귀적 한정적 타입

> 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정한다. <br>
주로 타입의 자연적 순서를 지정해주는 Comparable과 함께 사용된다.

```
public interface Comparable<T>{
	int compareTo(T o);
}
```

- 예시

```
public class ComparableBox<T extends Comparable<T>> {  // T는 Comparable<T>를 상속한 타입만 가능
    private T value;
    
    public ComparableBox(T value) {
        this.value = value;
    }

    // T 타입의 객체와 비교
    public int compareTo(T other) {
        return value.compareTo(other); // T 타입의 compareTo 메서드 호출
    }
}
```

```
public class Main {
    public static void main(String[] args) {
        ComparableBox<Integer> box1 = new ComparableBox<>(10);
        ComparableBox<Integer> box2 = new ComparableBox<>(20);

        System.out.println(box1.compareTo(box2)); // -1 (10 < 20)
    }
}
```

>  Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최솟값이나 최댓값을 구하는 식으로 사용된다. 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다. <br>
> → 따라서 재귀적 한정적 타입을 통해 타입 매개변수의 허용 범위를 한정한다.

### 결론

> 제네릭 타입 메서드가 타입 안전하며 사용하기 쉽다. 불필요한 형변환을 없애기 위해 메서드의 매개변수와 반환값에 적절히 제네릭을 사용하여 제네릭 메서드로 만들면 편의성과 안정성을 모두 잡는 좋은 방법이 될 수 있다.