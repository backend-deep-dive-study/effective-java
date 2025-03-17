# 자원을 직접 명시하지말고 의존 객체 주입을 사용하라
사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

```
public class MyClass {
		private static final ResourceType resource;
		private MyClass() {}  // 인스턴스화 방지
		//...
}
```
- 단순한 연산 수행에는 적합하지만, 상태를 가질 수 없고 확장성 부족
- 의존성 주입이 불가능하여 테스트 및 유지보수가 어려움


```
public class MyClass {
		private final ResourceType resource;
		private MyClass(...) {}
		public static MyClass INSTANCE = new MyClass(...);  // 싱글턴
		//...
}
```
- 하나의 인스턴스만 유지되어야 하는 경우엔 적합하지만,
- 강한 결합도를 초래하여 테스트 및 확장성이 떨어짐

```
public class MyClass {
		private ResourceType resource;
		// ResourceType에 의존하는 Myclass 클래스
		public MyClass(ResourceType resource) {  // 의존 객체 주입
				this.resource = Objects.requireNonNull(resource);
		}
		//...
}
```

- 장점
> 클래스의 유연성, 재사용성, 테스트 용이성을 엄청나게 개선한다. <br>
자원의 개수나 의존 관계에 영향을 받지 않는다. <br>
불변을 보장하여 해당 자원을 사용하는 여러 클라이언트가 안전하게 공유할 수 있다.
- 단점
> 의존성이 수 천 개나 되는 대형 프로젝트에서는 코드가 복잡해질 수 있다. <br>
→ 수천 개의 클래스가 서로 다양한 자원에 의존할때, 이를 모두 생성자를 통해 주입해야 한다면, 클래스 간의 의존관계를 파악하고 관리하는데 어렵다. <br>
→ Spring 같은 의존 객체 주입 프레임워크를 이용하면 해소 가능

> 결론적으로, **자원을 직접 명시하지 말고 의존 객체 주입을 사용하라** 는 의미는
new를 사용해서 객체를 직접 만들지 말고, 외부에서 주입받아라 그래야 유연하고 유지보수하기 좋은 코드가 된다는 뜻입니다.