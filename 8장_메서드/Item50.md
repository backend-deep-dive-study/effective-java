#  적시에 방어적 복사본을 만들라

> 클라이언트가 여러분의 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다.

### C/C++ vs Java

- 버퍼 오버런/오버플로우:
	> C/C++: 배열 경계를 넘어 쓰기가 가능해 인접 메모리 손상 <br>
	> Java: 경계 검사로 방지, 예외 발생


- 배열 오버런:
	> C/C++: 배열 범위를 벗어난 접근 가능 <br>
	> Java: 모든 배열 접근에 경계 검사 수행


- 와일드 포인터:
	> C/C++: 초기화되지 않거나 해제된 메모리를 가리키는 포인터 사용 가능 <br>
	> Java: 참조는 항상 유효한 객체를 가리키거나 null (NullPointerException 발생)


- 메모리 누수:
	> C/C++: 할당된 메모리를 해제하지 않으면 누수 발생 <br>
	> Java: 가비지 컬렉터가 자동으로 처리 (완전히 방지되지는 않지만 발생 가능성 감소)

### 예시
> 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하다. <br>
> 하지만 주의를 기울이지 않으면 자기도 모르게 내부를 수정하도록 허락하는 경우가 생긴다.
```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    // TOC(Time-Of-Check)
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }

    // TOU(Time-Of-Use)
    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }
}
```

### 첫번째 공격
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);
```
>  Date는 클래스 내부 필드가 가변이라서 의도와 다르게 값이 바뀔 수 있다.
> start() 메서드나 end() 메서드에서 start 필드와 end 필드가 반환 가능하고, 내부 메서드를 통해 값도 수정이 가능하다.

→ Date는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안된다. (LocalDateTime이나 ZonedDateTime을 사용)

### 방어(TOC)

- 외부 공격으로부터 내부를 보호하려면 생성자에서 받은 기본 매개변수 각각을 방어적으로 복사해야 한다.
```java
public Period(Date start, Date end) {
    // 방어적 복사로 외부 참조와의 연결을 끊음
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    // 복사본으로 유효성 검사
    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(this.start + " after " + this.end);
    }
}
```

> 매개변수의 유효성을 검사(아이템49)하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사해야한다 <br>
> 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.

### 두번째 공격

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);
```
>  Period 인스턴스는 아직도 변경 가능하다. 접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문이다.

### 방어(TOU)

- 가변 필드의 방어적 복사본을 반환한다.
```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```
> 모든 필드가 객체 안에 완벽하게 캡슐화되었다.

### 결론

> TOCTOU(Time-Of-Check/Time-Of-Use) 공격 방지: 유효성 검사와 복사 사이에 객체가 변경될 수 있는 위험을 방지하기 위해 먼저 복사본을 만들고 난 후 유효성 검사를 수행합니다. <br>
> clone() : clone() 메서드는 하위 클래스에서 오버라이드될 수 있어 항상 안전하지 않습니다. 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone을 사용해도 된다. <br>
> 불변 객체 사용: 가능하면 Date 대신 LocalDateTime과 같은 불변 객체를 사용하는 것이 더 안전합니다. <br>
> 성능 고려: 방어적 복사는 비용이 발생하므로, 신뢰할 수 있는 코드 내에서 사용되는 객체라면 문서화를 통해 복사 없이 사용할 수 있습니다. <br>

---

# 부록

### Java의 메모리 안정성

- 자동 메모리 관리:
	> 가비지 컬렉터가 더 이상 사용되지 않는 객체를 자동으로 정리 <br>
	> 프로그래머가 직접 메모리를 할당/해제하지 않아도 됨


- 경계 검사:
	> 배열이나 문자열 접근 시 항상 경계 검사 수행 <br>
	> 배열 인덱스가 유효 범위를 벗어나면 ArrayIndexOutOfBoundsException 발생


- 포인터 대신 참조:
	> Java는 명시적인 포인터를 사용하지 않고 참조를 사용 <br>
	> 참조는 항상 유효한 객체를 가리키거나 null <br>
	> 와일드 포인터나 댕글링 포인터 문제가 없음


- 타입 안전성:
	> 강력한 타입 시스템으로 잘못된 타입 변환 방지 <br>
	> 컴파일 시간과 실행 시간 모두에서 타입 검사