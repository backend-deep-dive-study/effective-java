# 톱레벨 클래스는 한 파일에 하나만 담으라

> 소스 파일 하나에 톱레벨 클래스를 여러개 사용하더라도 컴파일러는 문법적으로 막지않지만, 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위다.

## 예제

Main.java
```
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    } 
}
```
Utensil.java (pancake)
```
class Utensil {
		static final String NAME = "pan";
}

class Dessert {
		static final String NAME = "cake";
}
```

Dessert.java (potpie)
```
class Utensil {
		static final String NAME = "pot";
}

class Dessert {
		static final String NAME = "pie";
}
```

- 컴파일 순서 Main -> Dessert

> 1. 메인을 먼저 컴파일
> 2. 그 안에서 Utensil.java를 살피면서 Utensil과 Dessert를 모두 찾아낸다.
> 3. 명령으로 넘어온 Dessert.java를 처리할 때 오류 발생생

- 컴파일 순서 Utensil -> Main

> 1. Utensil를 먼저 컴파일
> 2. main함수에서 pancake 출력

- 컴파일 순서 Dessert -> Main

> 1. Dessert를 먼저 컴파일
> 2. main함수에서 potpie 출력

## 해결책

> 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용 <br>
다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 더 나음 읽기 좋고, private로 선언하면 접근 범이도 최소로 관리할 수 있음

```
public class Test {
	public static void main(String[] args) {
			System.out.println(Utensil.NAME + Dessert.NAME);
	}
		
	private static class Utensil {
			static final String NAME = "pan";
	}
		
	private static class Dessert {
			static final String NAME = "cake";
	}
}
```