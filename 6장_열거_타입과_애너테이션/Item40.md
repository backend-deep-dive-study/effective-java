#  @Override 애너테이션을 일관되게 사용하라

> 자바가 기본으로 제공하는 애너테이션 중 보통의 프로그래머 에게 가장 중요한 것은 @Override일 것이다. <br>
@Override는 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다.<br>
→ 이 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.

```
// 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그 존재
import java.util.*;

public class Bigram {
	private final char first;
	private final char second;
	
	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}
	// 문제가 되는 부분
	public boolean equals(Bigram b) {
		return b.first == first && b.second == second;
	}
	
	public int hashCode() {
		return 31 * first + second;
	}
	
	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();
		for(int i = 0; i < 10; i++) {
			for(char ch = 'a'; ch <= 'z'; ch++) {
				s.add(new Bigram(ch,ch));
			}
		}
		System.out.println(s.size());
	}
}
```
### 코드 설명
> 이 코드는 main 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력한다. Set은 중복을 허용하지 않으니 26이 출력될 거 같지만, 실제로는 260이 출력된다.

### 잘못된 점
> equals 메서드를 재정의(overriding)하려 했고(아이템 10), hashCode도 함께 재정의 했다(아이템 11). 하지만 equals 메서드를 재정의한 것이 아니라 다중정의의(overloading, 아이템 52)해버렸다.

→ Object의 equals를 재정의하려면 매개변수타입을 Object로 해야만 하는데, 그렇게 하지 않았다. 그래서 Object에서 상속한 equals와는 별개인 equals를 새로 정의한 꼴이 되었다.

### 재정의(Override)와 다중정의의(Overload)의 차이:
- 오버라이딩: 상위 클래스의 메서드를 같은 시그니처로 재정의
- 오버로딩: 같은 이름이지만 다른 매개변수를 가진 새로운 메서드 정의

### 수정

> @Override 애너테이션을 달고 다시 컴파일하면 다음의 컴파일 오류가 발생한다. <br>
잘못한 부분을 명확히 알려주므로 곧장 올바르게 수정할 수 있다.


```
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram)) return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```

- 예외 : 구체 클래스에서 상위 클래스의 추상메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다.

### 결론
> 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 실수했을 때 컴파일러가 바로 알려줄 것이다. 예외는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지 않아도 된다(단다고 해서 해로울 것도 없다).