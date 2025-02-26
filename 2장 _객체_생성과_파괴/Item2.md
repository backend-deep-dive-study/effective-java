#  생성자에 매개변수가 많다면 빌더를 고려하라
## 객체 생성 패턴
### 1. 점층적 생성자 패턴(Telescoping Constructor Pattern)
```java
public NutritionFacts(int servingSize, int servings) { 
  this(servingSize, servings, 0); // 다음 생성자 호출
}
public NutritionFacts(int servingSize, int servings, int calories) { 
  this(servingSize, servings, calories, 0);
}
```
#### 장점
- 객체 생성 시 한 번의 호출로 불변 객체 생성
- 모든 값을 한 번에 설정하므로 일관성 유지 가능
- 매개 변수에 대한 유효성 검사를 생성자에서 한 번에 처리 가능
#### 단점
- 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려움
- 타입이 같은 매개변수가 연달아 있으면 순서를 바꿔도 컴파일러가 알아채지 못함
- 찾기 어려운 버그로 이어질 수 있음
- 선택적 매개변수가 많은 경우, 필요하지 않은 매개변수에도 값을 지정해야 함
### 2. 자바빈즈 패턴(JavaBeans Pattern)
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
// 필요한 setter 메서드 호출
```
#### 장점
- 객체 생성이 더 읽기 쉽고 이해하기 쉬움
- 매개변수가 많아도 각 값의 의미를 명확히 알 수 있음
- 선택적 매개변수에 대해 유연하게 처리 가능(원하는 값만 설정)
- 기존 객체의 값 변경이 쉬움
#### 단점
- 객체 하나를 만들기 위해 여러 메서드 호출 필요
- 객체가 완전히 생성되기 전까지 일관성이 무너진 상태
- 클래스를 불변으로 만들 수 없음
- 스레드 안정성을 위해 추가 작업 필요
- 버그를 디버깅하기 어려움(객체 생성 코드와 문제가 발생하는 코드가 물리적으로 떨어져 있기 때문)
### 3. 빌더 패턴(Builder Pattern)
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();
```
#### 장점
- 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비
- 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출 가능 (메서드 연쇄/플루언트 API)
- 불변 객체 생성 가능
- 빌더의 생성자와 메서드에서 입력 매개변수를 검사할 수 있고, build메서드가 호출하는 생성자에서 불변식(invariant)을 검사할 수 있음
- 계층적으로 설계된 클래스와 함께 쓰기에 좋음(각 계층의 클래스에 관련 빌더를 멤버로 정의)
- 빌더 하나로 여러 객체를 순회하며 만들 수 있음
- 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수 있음
- 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수 있음
- 가변인수(varargs) 매개변수를 여러 개 사용할 수 있음
#### 단점
- 객체를 만들기 위해서 빌더부터 만들어야 함.
- 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있음
- 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 함


### 정리
- 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 것이 좋음
- 특히 매개변수 중 다수가 필수가 아니거나 같은 타입일 때 유용
- 빌더는 점층적 생성자보다 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전함
- API는 시간이 지날수록 매개변수가 많아지는 경향이 있으므로, 애초에 빌더로 시작하는 편이 나을 때가 많음

 
<hr>

## 부록
### 점층적 생성자 패턴이란?
- 일반 생성자 패턴의 특수한 형태로, 생성자들이 서로 계층적으로 호출하는 구조를 가짐. 
<br> 즉, 매개변수가 적은 생성자가 매개변수가 많은 생성자를 호출하는 방식으로 설계됨. 
### 클라이언트 코드란?
- API나 클래스를 직접 구현하는 코드가 아니라, 그것을 사용하는 측의 코드.
### 일관성이 무너진 상태란? (자바빈즈 패턴의 단점 2번)
- 객체가 생성되는 과정에서 완전하지 않은 중간 상태가 발생한다는 의미.
```java
NutritionFacts cocaCola = new NutritionFacts();  // 1. 기본 생성자로 객체 생성
cocaCola.setServingSize(240);                    // 2. 필수 값 설정
cocaCola.setServings(8);                         // 3. 필수 값 설정
cocaCola.setCalories(100);                       // 4. 선택 값 설정
cocaCola.setSodium(35);                          // 5. 선택 값 설정
// ... 더 많은 setter 호출 가능
```
#### 일관성이 무너진 상태에서 발생하는 문제점들
1. 불완전한 중간 상태
- 모든 필수 필드가 설정되기 전에 객체가 이미 생성되어 있음. 
<br>servingSize와 servings가 필수 값인 경우, 1번 단계 후 객체는 유효하지 않은 상태.
2. 값 검증의 어려움 
- 점층적 생성자 패턴에서는 생성자 내에서 모든 매개변수의 유효성을 한 번에 검사할 수 있지만, 
<br> 자바빈즈 패턴에서는 각 setter 메서드에서 개별적으로 검사해야 함. -> 여러 필드 간의 관계에 기반한 복합적인 유효성 검사가 어려워지게 됨.
3. 불변성 보장 불가
- 모든 setter 호출이 완료되기 전까지는 객체가 예상대로 동작하지 않을 수 있음. 
<br> 그리고 setter 메서드가 있는 한 객체는 언제든 변경될 수 있으므로 불변 객체로 만들 수 없음.
4. 스레드 안전성 문제
- 여러 스레드가 동시에 하나의 객체에 대해 setter 메서드를 호출하면 경쟁 상태(race condition)가 발생할 수 있음.
### 메서드연쇄(Method Chaining)란?
- 객체의 메서드가 자기 자신의 객체를 반환하게 함으로써, 하나의 문장에서 여러 메서드 호출을 연속해서 할 수 있게 해주는 기법.
```java
public class Builder {
    // 메서드가 this를 반환하여 메서드 체이닝 가능
    public Builder methodA() {
        // 내부 로직
        return this;
    }
    
    public Builder methodB() {
        // 내부 로직
        return this;
    }
    
    public Object build() {
        return new Object();
    }
}

// 메서드 연쇄 없이
Builder builder = new Builder();
builder.methodA();
builder.methodB();
Object obj = builder.build();

// 메서드 연쇄 사용
Object obj = new Builder()
    .methodA()
    .methodB()
    .build();
```
### 플루언트 API(Fluent API)란?
- 메서드 연쇄를 기반으로 하되, 더 나아가 API가 마치 자연어처럼 읽히도록 메서드 이름과 호출 순서를 설계하는 방식
```java
// SQL 빌더 예시
// 일반적인 방식
Query query = new Query();
query.select("name", "email");
query.from("users");
query.where("age > 18");
ResultSet results = query.execute();

// 플루언트 API
ResultSet results = new QueryBuilder()
    .select("name", "email")
    .from("users")
    .where("age > 18")
    .execute();
```
