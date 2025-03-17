# equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

→ 재정의하지 않을 경우 hashCode의 일반 규약을 어기게 되어 HashMap, HashSet의 원소로 사용할 때 문제 발생

### HashCode 일반 규약

1. equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode는 변하지 않고 항상 일관된 값을 반환. <br>
   (애플리케이션 다시 실행 시 달라질 수 있음)
2. equals(Object)가 두 객체를 같다고 판단하면, 두 객체의 hashCode는 똑같은 값 반환.
3. equals(Object)가 두 객체를 다르다고 판단해도, 두 객체의 hashCode가 꼭 다른 값을 반환할 필요 없음. <br>
   but 다른 객체에 대해서 다른 값을 반환해야 해시테이블의 성능이 좋아짐.
    - 다른 값을 반환할 필요가 없는 이유 ~~(문제는 없지만 성능 저하)~~
        - hashCode : 해시테이블에 객체를 저장할 때 버킷(같은 hashCode를 가진 객체를 저장하는 공간)을 결정
        - equals()가 false를 반환하는 두 객체라도 충돌이 발생하지만 같은 hashCode 가질 수 있음
    - 다른 code를 반환해야 성능이 좋아지는 이유
        - 같은 버킷에 있는 객체들만 equals()로 비교
        - 서로 다른 객체들이 다른 hashCode를 가지면 버킷이 균등하게 분산되어 검색 속도가 빨라짐
        - hashCode가 항상 같은 값을 반환할 경우, 모든 객체가 같은 버킷에 몰려 hashCode 충돌이 많이 발생 → equals() 비교 연산이 많이 발생하여 성능 저하

### 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다

- equals()는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있음
- Object의 기본 hashCode는 위와 같은 상황에 둘이 다르다고 판단하여 다른 값 반환

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```

- m.get(new PhoneNumber(707, 867, 5309)) 실행 시 null 반환

⇒ PhoneNumber 클래스가 hashCode를 재정의하지 않아 논리적 동치인 두 객체가 서로 다른 hashCode를 반환하였기 때문 (2번 규약 깨짐)

### 동치인 모든 객체에 똑같은 hashCode 반환

```java
public int hashCode() {
	return 30;
}
```

- 동치인 모든 객체에 똑같은 hashCode를 반환하므로 적법
- 하지만 모든 객체가 해시테이블의 버킷 하나에 담겨 연결리스트처럼 동작
    - 평균 수행 시간이 O(1) → O(n)으로 느려짐

### 좋은 hashCode 작성 및 사용

- hashCode 작성 요령 (교재 p.70 참고)
- hashCode 캐싱
    - 클래스가 불변이고 해시코드를 계산하는 비용이 클 경우, 매번 계산하기 보다는 캐싱하기
    - 해당 타입의 객체가 주로 해시의 키로 사용될 것 같으면 인스턴스가 만들어질 때 해시코드를 계산해둠
- 지연 초기화
    - 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 호출될 때 계산하도록 지연 초기화
    - 필드 지연 초기화 시 해당 클래스가 thread-safe가 되도록 동기화에 신경 써야함
- 해시코드 계산 시 핵심 필드 생략하지 않기
    - 속도는 빨라질 수 있지만 해시 품질 저하 → 해시테이블 성능 저하 (hashCode가 버킷에 고르게 분포되지 않는 문제)
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말기
    - 클라이언트가 해당 값에 의존하지 않고, 후에 계산 방식을 바꿀 수 있도록

### 결론

- equals 정의 시 hashCode를 반드시 재정의
    - 하지 않으면 프로그램이 제대로 동작하지 않음
- 재정의한 hashCode는 Object API 문서에 있는 일반 규약 지키기
- 서로 다른 인스턴스라면 hashCode도 서로 다르게 구현하기
    - ~~아이템10에 나온 것 처럼 AutoValue 프레임워크 사용 시 equals와 hashCode 자동 생성해준다~~
