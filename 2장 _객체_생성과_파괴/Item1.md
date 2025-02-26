# 생성자 대신 정적 팩터리 메서드를 고려해라

### 클래스 인스턴스 얻기

- public 생성자
- 정적 팩터리 메서드 (해당 클래스의 인스턴스를 반환하는 단순 정적 메서드)
    
    ex) 기본 타입인 boolean을 Boolean 객체로 변환해주는 valueOf 메서드
    
    ```java
    public static Boolean valueOf(boolean b){
    	return b ? Boolean.TRUE : Boolean.FALSE;
    }
    ```
    

### 정적 팩터리 메서드 장점 ~~(생성자보다 좋은 이유)~~

- 이름을 가질 수 있다
    - 이름으로 인스턴스가 가지는 특성을 알 수 있음
    - 생성자는 매개변수의 종류, 개수, 순서가 같을 시 중복 생성 불가
    - 시그니처가 같은 생성자가 여러 개 필요할 경우, 생성자를 정적 팩터리 메서드로 바꾸고 이름에 특성 묘사
- 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다
    - 인스턴스 통제 클래스임 (언제 어느 인스턴스를 살아 있게 할지 통제)
    - 생성자를 사용할 경우 같은 요청이 반복되어도 항상 새로운 객체 생성 후 반환
    
    ```java
    public class UserSession { //세션 유지 안됨
        private final String sessionId;
    
        public UserSession(String sessionId) {
            this.sessionId = sessionId;
        }
    }
    ```
    
    - 정적 팩터리 메서드 이용 시 인스턴스 캐싱하여 재활용 가능
    
    ```java
    public class UserSession { // 항상 동일한 인스턴스를 반환해 세션 유지 가능
        // 캐싱하는 맵
        private static final Map<String, UserSession> SESSION_CACHE = new ConcurrentHashMap<>();
    
        private final String sessionId;
    
    		// 생성자 private -> 해당 클래스 내에서만 사용
        private UserSession(String sessionId) {
            this.sessionId = sessionId;
        }
    
        // 정적 팩토리 메서드를 사용하여 동일한 sessionId에 대해 같은 인스턴스를 제공
        public static UserSession getSession(String sessionId) {
            return SESSION_CACHE.computeIfAbsent(sessionId, UserSession::new);
        }
    }
    ```
    
    - 플라이웨이트 패턴이란?
        
        객체를 효율적으로 공유하여 메모리 사용을 최적화하는 디자인 패턴
        
        여러 객체가 동일한 상태를 공유하여 불필요한 중복 객체를 만들지 않도록 하여 메모리 절약
        
        → 해당 패턴에서 공유 객체 관리 시 정적 팩터리 메서드 사용하여 객체 생성&반환
        
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
    - 반환 타입을 부모 클래스 or 인터페이스로 설정 가능
    - 구현 클래스를 공개하지 않을 수 있음
    - API 작게 유지 가능 (변경이 적음 + 코드와 인터페이스가 간결 + 확장이 쉬운 구조)
    
    ```java
    public static void main(String[] args) {
        // 구체적인 클래스가 메인에 직접 노출됨
        Book book = new Book("Effective Java");
        Laptop laptop = new Laptop("MacBook Pro");
        Smartphone smartphone = new Smartphone("iPhone 15");
    }
    ```
    
    ```java
    abstract class Product {
        protected String name;
    
        public Product(String name) {
            this.name = name;
        }
    
        public abstract String description();
    
        // 팩토리 메서드로 객체 생성
        public static Product createBook(String title) {
            return new Book(title);
        }
    
        public static Product createLaptop(String model) {
            return new Laptop(model);
        }
    
        public static Product createSmartphone(String brand) {
            return new Smartphone(brand);
        }
    }
    
    public static void main(String[] args) {
        // 반환 타입을 부모로 설정하여 구체적인 클래스가 메인에 노출되지 않음
        Product book = Product.createBook("Effective Java");
        Product laptop = Product.createLaptop("MacBook Pro");
        Product smartphone = Product.createSmartphone("iPhone 15");
    }
    ```
    
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    - 반환 타입의 하위 타입이면 어떤 클래스의 객체를 반환 가능
    
    ```java
    public interface Rate {
        int score();
        
        // 정적 팩토리 메서드: score에 따라 다른 클래스 객체 반환
        static Rate from(int score) {
            if (score >= 2400) {
                return new Challenger(score);
            }
        
            if (score >= 1900) {
                return new Diamond(score);
            }
        
            return new Silver(score);
        }
        
        default String description() {
            return String.format("%s : %d", this.getClass().getSimpleName(), score());
        }
    }
    ```
    
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    - createShape은 실제 구현 클래스를 반환하는 정적 팩토리 메서드
        
        → Circle과 Square의 클래스는 존재해야 하지만 반환할 객체는 호출 시점에 동적으로 결정
        
    
    ```java
    interface Shape {
        void draw();
    }
    
    class Circle implements Shape {
        @Override
        public void draw() {
            System.out.println("Drawing Circle");
        }
    }
    
    class Square implements Shape {
        @Override
        public void draw() {
            System.out.println("Drawing Square");
        }
    }
    
    class ShapeFactory {
        public static Shape createShape(String type) {
            if ("circle".equalsIgnoreCase(type)) {
                return new Circle();
            } else if ("square".equalsIgnoreCase(type)) {
                return new Square();
            }
        }
    }
    ```
    

### 정적 팩터리 메서드 단점

- 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
    - 정적 팩터리 메서드를 구현한 클래스는 상속 불가
        
        → 해당 메서드 사용을 유도하기 위해 생성자를 private로 설정하기 때문
        
- 프로그래머가 찾기 어렵다
    - 생성자는 클래스명과 동일하기에 구분이 쉽지만 개발자가 이름을 정한 정적 팩터리 메서드의 경우 다른 메서드들과 섞여 찾기 힘듦
    - 정적 팩터리 메서드 이름 규약 존재
        
        → from, of, valueOf, create 혹은 newInstance, instance 혹은 getInstance, getType, newType, type
        

### 결론

- 한 클래스에 같은 시그니처의 생성자가 여러 개 필요한 경우
- 반복되는 요청에 인스턴스 재활용이 가능할 경우
- 매개변수 값에 따라 다른 값을 반환하고 싶은 경우

⇒ 생성자보다 정적 팩터리 메서드를 사용하자
