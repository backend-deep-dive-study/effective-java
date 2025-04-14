# 전통적인 for문 보다는 for-each문을 사용하라

### 전통적인 for문의 문제점

```java
//컬렉션 순회
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  System.out.println(e);
}

//배열 순회
for (int i = 0; i < a.lenght; i++) {
  System.out.println(a[i]);
}
```

1. 오류가 생길 가능성이 높아짐
2. 변수를 잘못 사용할 가능성이 높아짐
3. 잘못된 변수를 썼을 때 컴파일러에 잡히지 않을 수 있음
4. 컬렉션이냐 배열이냐에 따라 코드 형태가 달라짐

### 문제 해결 -> for-each문 사용

- for-each문 = 향상된 for문 (enhanced for statement)
- 반복자, 인덱스 변수를 사용하지 않아 코드가 깔끔하고 오류가 날 일이 적음
- 하나의 관용구로 컬렉션과 배열을 처리할 수 있어 어떤 컨테이너를 다루는지 신경 쓰지 않아도 됨

```java
for (Element e : elements) {
  System.out.println(e);
}
```

- ':' 콜론은 '안의(in)'이라고 읽으면 됨 -> elements 안의 각 원소 e에 대해
- 속도는 for문과 동일함

### 컬렉션의 중첩 순회

- 반복문 중첩 시 자주 발생하는 문제
- 바깥 컬렉션의 반복자가 안쪽 반복문에서 호출되는 경우 문제가 생김
- 해결 : 바깥 반복문에 바깥 원소를 저장하는 변수 추가

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext();) {
  Suit suit = i.next();
  for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
    deck.add(new Card(suit, j.next()));
}
```

- 하지만 for-each문을 중첩하면 더 간단히 해결됨

```java
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit, rank));
```

### for-each문을 사용할 수 없는 상황

1. 파괴적인 필터링

- 컬렉션을 순회하면서 선택된 원소를 제거해야 할 때 반복자의 remove 메서드 호출
- 자바8부터는 Collection의 removeIf를 사용해 명시적으로 순회하지 않을 수 있음

2. 변형

- 리스트나 배열을 순회하면서 원소의 값 일부 혹은 전체를 교체할 때 리스트의 반복자나 배열의 인덱스를 사용해야 함

3. 병렬 반복

- 여러 컬렉션을 병렬로 순회해야 할 때 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 함

### for-each문이 순회할 수 있는 컨테이너

- 컬렉션, 배열
- Iterable 인터페이스를 구현한 객체
- 원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현해볼 것!

### 결론 : 가능하면 for문 대신 for-each문을 사용하자!
