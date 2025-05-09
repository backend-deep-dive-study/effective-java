# 필요 없는 검사 예외 사용은 피하라

### 검사 예외의 단점

- 과하게 사용 시 쓰기 불편한 API가 됨
    - catch로 예외를 붙잡아서 처리
    - 바깥으로 던져 문제 전파
- 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없음

### 검사 예외 사용

- API를 제대로 사용해도 발생할 수 있는 예외
- 의미 있는 조치를 취할 수 있는 경우
- 더 나은 예외 처리 방법이 없다면 비검사 예외 사용
    
    ```java
    // API 설계상 검사 예외를 선언 했지만 실제로 절대 던지지 않는 경우
    } catch (TheCheckedException e) {
    	// 컴파일러는 예외가 발생할 수 있다고 인식
    	// 개발자의 절대 발생 안한다는 판단이 틀렸을 경우 프로그램 비정상 종료 -> 예외 복구 불가능
    	throw new AssertionError();
    }
    ```
    
    ```java
    // 검사 예외는 복구 가능성이 있는 예외를 의미 -> 스택 트레이스 출력 후 프로그램 강제 종료
    } catch (TheCheckedException e) {
    	e.printStackTrace();
    	System.exit(1);
    }
    ```
    

### 검사 예외 회피 방법: 옵셔널 사용

- 적절한 결과 타입을 담은 옵셔널 반환
- 검사 예외 대신 빈 옵셔널 반환
- 단점) 예외가 발생한 이유를 알려주는 부가 정보를 담을 수 없음

### 검사 예외 회피 방법: 메서드 분리

- 검사 예외를 던지는 메서드를 2개로 나눠 비검사 예외로 바꿈
- 첫 번째 메서드는 예외가 던져질지 여부를 boolean으로 반환

#### 검사 예외를 던지는 메서드

- 리팩터링 전

```java
try {
	obj.action(args);
} catch (TheCheckedException e) {
	... // 예외 상황에 대처한다.
}
```

#### 상태 검사 메서드와 비검사 예외를 던지는 메서드

- 리팩터링 후

```java
if (obj.actionPermitted(args)) { // 상태 검사 메서드
	obj.action(args);
} else {
	... // 예외 상황에 대처한다.
}
```

- 이 리팩터링은 모든 상황에 적용할 수 없지만 적용할 수 있으면 쓰기 편한 API 제공 가능

#### 그 외

- 프로그래머가 해당 메서드의 성공을 아는 경우
- 실패 시 스레드 중단을 원할 경우

```java
obj.action(args); 
```

- 이렇게 자주 쓰인다고 판단되면 리팩터링하는 게 좋음

#### actionPermitted

- 상태 검사 메서드
- 리팩터링 적절X
    - 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있는 경우
        - actionPermitted와 action 호출 사이에 객체의 상태가 변할 수 있기 때문
    - 만약 actionPermitted가 action 메서드의 작업 일부를 중복 수행한다면 성능에서 손해

### 정리

- API 호출자가 예외 상황에서 복구할 방법이 없다면 비검사 예외 사용
- 복구 가능하고 호출자가 그 처리를 해주길 원하면 옵셔널부터 고민
- 옵셔널만으로 상황을 처리하기에 충분한 정보를 제공할 수 없으면 검사 예외 사용
