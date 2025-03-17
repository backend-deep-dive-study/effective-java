# 비검사 경고를 제거하라
## 제네릭 사용 시 발생하는 경고 유형
- 비검사 형변환 경고
  - 제네릭 타입을 명확히 지정하지 않을 때 발생
- 비검사 메서드 호출 경고
  - 제네릭을 사용하는 메서드 호출 시 발생
- 비검사 매개변수화 가변인수 타입 경고
  - 가변 인자(varargs)와 제네릭을 함께 사용할 때 발생
- 비검사 변환 경고
  - 원시 타입(raw type)에서 제네릭 타입으로 변환할 때 발생
## 비검사 경고 제거 방법
### 1. 타입 매개변수 명시
- 대부분의 경고는 적절한 타입 매개변수를 지정하여 제거 가능
```java
// 컴파일러가 경고!
Set<Lark> exaltation = new HashSet();

// 자바 7부터 지원하는 다이아몬드 연산자(<>)만으로 해결 가능!
// 컴파일러가 올바른 실제 타입 매개변수를 추론해줌.
Set<Lark> exaltation = new HashSet<>();
```
### 2. @SupressWarnings("unchecked") 사용
- 경고를 제거할 수 없지만 타입 안전성이 보장되는 경우, 애너테이션을 활용해 경고를 숨길 수 있음.
- 단, 무분별한 사용은 위험하며, 반드시!! 타입 안전성을 검토한 후 최소 범위에 적용해야 함.
- SuppressWarnings를 사용할 경우 그 경고를 무시해도 안전한 이유를 항상 주석으로 남기자.
  - 다른 개발자의 이해를 돕고 코드가 잘못 수정되는 위험을 줄임.
## @SupressWarnings 적용 범위 최소화
- 클래스 전체 또는 메서드가 아닌, 지역 변수나 아주 짧은 메서드 단위로 적용
- 나쁜 예시
```java
@SuppressWarnings("unchecked") // 클래스 전체에 적용
public class UnsafeList {
    private List rawList = new ArrayList(); // 원시 타입 사용 (경고 발생 X, IDE에 따라 경고를 줄 수도 있음음)

    public void addItem(Object item) {
        rawList.add(item); // 경고 발생 X
    }

    public List getList() {
        return rawList; // 타입 불안전한 리스트 반환
        // 사용자 코드에서 ClassCastException 발생 가능
    }
}
```
- 좋은 예시
```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // 생성한 배열과 매개변수로 받은 배열의 타입이 같으므로 올바른 형변환
        @SuppressWarnings("unchecked") 
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) a[size] = null;
    return a;
}
```
## 비검사 경고 처리의 중요성
- 모든 비검사 경고를 제거하면 타입 안전성이 보장됨.
- 타입 안전성이 보장되면 런타임에 ClassCastException이 발생하지 않음.
- 비검사 경고를 무시하면 새로운 경고를 발견하기 어려워질 수 있음.
  - 제거하지 않은 수많은 거짓 경고에 새로운 경고가 파묻히기 때문.
## 핵심 정리
- 비검사 경고는 중요하므로 무시하지 말 것
- 모든 비검사 경고는 런타임에 ClassCastException을 발생시킬 가능성이 있음
- 최선을 다해 경고를 제거할 것
- 제거할 수 없다면 타입 안전함을 증명하고 최소 범위에 @SuppressWarnings("unchecked")를 적용
- 경고를 숨긴 근거를 주석으로 반드시 남길 것
