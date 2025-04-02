# null이 아닌, 빈 컬렉션이나 배열을 반환하라

## null을 반환하는 경우

```java
// 컬렉션이 비어있으면 null을 반환하는 코드 - 절대 따라하지 말 것!

private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 *         단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArryaList<>(cheesesInStock);
}
```

- 이 코드처럼 null을 반환하면 클라이언트는 null을 처리하는 코드를 추가로 작성해야 한다.

    ```java
    List<Cheese> cheeses = shop.getCheeses();
    if (cheeses != null && cheeses.contains(Cheese.SILTON))
        System.out.println("좋았어, 바로 그거야.);
    ```

    - 이런 식으로 클라이언트에 방어 코드를 넣어줘야 한다.

    - 클라이언트가 까먹게 된다면 오류가 발생할 수 있다.

    - 추가로 null을 반환하는 쪽에서도 특별 취급을 해야 하기 때문에 코드가 더 복잡해진다.

## 빈 컬렉션을 반환하는 경우

- `new ArrayList<>(cheesesInStock);`과 같이 빈 컬렉션을 만들어서 주어도 되지만, 비어있는 '불변' 컬렉션을 반환하는 것이 더 좋다.

    - `Collections.emptyList`, `Collections.emptySet`, `Collections.emptyMap`

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArryaList<>(cheesesInStock);
}
```

## 빈 배열을 반환하는 경우

- 배열도 마찬가지로 null을 반환하지 말고 길이가 0인 배열을 반환해야 한다.

- `cheesesInStock.toArray(new Cheese[0])`으로 해주어도 좋지만 길이 0짜리 배열을 미리 선언해두는 것이 좋다.

    - 길이 0인 배열은 모두 불변이기 때문이다.

    ```java
    private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

    public Cheese[] getCheeses() {
        return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
    }
    ```

- 단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다. 성능이 떨어진다는 연구 결과도 존재하기 때문이다.

    ```java
    return cheesesInStock.toArray(new Cheeses[cheesesInStock.size()]);
    ```

## 결론

- **null이 아닌, 빈 배열이나 컬렉션을 반환하라.**

- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다.

- 그렇다고 성능이 좋은 것도 아니다.