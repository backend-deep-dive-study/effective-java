#  스트림은 주의해서 사용하라

> 스트림(Stream) API는 Java 8에서 도입된 기능으로, 데이터 컬렉션을 선언형(함수형 스타일)으로 처리할 수 있도록 도와주는 API입니다.

### 추상적 개념

`스트림` : 데이터의 연속적인 흐름
 - 스트림의 원소는 다양한 데이터 소스(컬렉션, 배열, 파일 등)에서 오는 객체 참조나 기본 타입(int, long, double)

`스트림 파이프라인` : 스트림의 원소로 수행하는 연산 단계
- 스트림 생성 → 리스트, 배열, 파일 등에서 스트림 생성
- 중간 연산 → `filter()`, `map()` 등의 연산 적용
- 종단 연산 → `collect()`, `forEach()`, `count()` 등으로 결과 반환

### 스트림 파이프라인
- 스트림 파이프라인은 지연평가된다.

	> 스트림에서 중간 연산을 여러 개 호출해도, 실제로 데이터가 처리되지 않고 대기 상태에 있다가 최종 연산이 실행될 때 한꺼번에 처리됩니다. 
	<details>
	  <summary>지연 평가(Lazy evaluation)</summary>
		
		종단 연산(Terminal Operation)이 호출될 때까지 중간 연산(Intermediate Operation)이 실행되지 않는 것을 의미합니다.
	    
	</details>
- 스트림 API는 메서드 체이닝을 지원하는 플루언트 API다.
	> 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다. <br>
 	> 기본적으로 스트림 파이프라인은 순차적으로 수행된다. 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다(아이템 48).
	<details>
	  <summary>플루언트 API(Fluent API)</summary>
		
		Fluent API는 메서드를 연쇄적으로 호출할 수 있도록 설계된 API 스타일입니다.
		즉, 객체.메서드().메서드().메서드(); 형태로 연속적인 메서드 호출이 가능합니다.
	    
	</details>
- 스트림 API는 다재다능하여 사실상 어떠한 계산이라도 해낼 수 있다.
  	> 하지만 할 수 있다는 뜻이지, 해야 한다는 뜻은 아니다. 스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다.

### 주의할 점

1. 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.(가독성이 떨어짐)
	```
	// 복잡한 조건을 포함한 스트림 (가독성 저하)
	List<Employee> result = employees.stream()
		.filter(e -> e.getDepartment().equals("IT") && e.getSalary() > 5000 
				&& e.getYearsOfService() > 3 && !e.isOnLeave())
		.sorted(Comparator.comparing(Employee::getSalary).reversed()
				.thenComparing(Employee::getYearsOfService))
		.limit(10)
		.collect(Collectors.toList());

	// 더 명확한 명령형 코드
	List<Employee> filteredEmployees = new ArrayList<>();
	for (Employee e : employees) {
		if (e.getDepartment().equals("IT") && e.getSalary() > 5000 
				&& e.getYearsOfService() > 3 && !e.isOnLeave()) {
			filteredEmployees.add(e);
		}
	}
	filteredEmployees.sort(Comparator.comparing(Employee::getSalary).reversed()
			.thenComparing(Employee::getYearsOfService));
	List<Employee> result = filteredEmployees.subList(0, Math.min(10, filteredEmployees.size()));
	```
	> 복잡한 필터링 조건과 정렬 로직이 결합된 경우, 스트림 파이프라인은 오히려 코드를 읽기 어렵게 만들 수 있다. 명령형 접근 방식은 단계를 분리하여 보여주므로 각 단계가 무엇을 하는지 더 명확하게 이해할 수 있다.

2. char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.
	```
	// 좋지 않은 방식: char 처리에 스트림 사용
	"Hello".chars().forEach(x -> System.out.print((char) x));

	// 더 나은 방식: 전통적인 for 루프 사용
	for (char c : "Hello".toCharArray()) {
		System.out.print(c);
	}
	```
	> `chars()` 메서드는 `IntStream`을 반환하기 때문에 매번 `char`로 형변환해야 합니다. 이는 코드를 더 복잡하게 만들고 가독성을 떨어뜨립니다. 전통적인 반복문이 이 경우 더 간결하고 명확하하다.

3. 함수 객체(람다, 메서드 참조)로 표현할 수 없는 경우
	```
	// 스트림으로 표현하기 어려운 경우: 재귀적 순회
	public void traverseTree(TreeNode node) {
		if (node == null) return;
		// 노드 처리
		System.out.println(node.value);
		traverseTree(node.left);
		traverseTree(node.right);
	}
	```
	> 재귀적 알고리즘이나 복잡한 제어 흐름을 가진 로직은 함수형 방식으로 표현하기 어렵습니다. 트리 순회와 같은 경우, 전통적인 명령형 프로그래밍 방식이 더 자연스럽고 이해하기 쉽다.

4. 한 데이터를 파이프라인 여러 단계에서 접근이 필요할 때

	```
	// 스트림 파이프라인에서는 중간 결과에 접근하기 어려움
	Order order = ...; // 주문 정보

	// 더 명확한 명령형 방식
	double sum = 0;
	List<OrderItem> items = order.getItems();
	for (OrderItem item : items) {
		sum += item.getPrice() * item.getQuantity();
	}
	double discount = sum > 10000 ? sum * 0.1 : 0;
	double total = sum - discount;
	System.out.println("합계: " + sum + ", 할인: " + discount + ", 총액: " + total);
	```
	
	> 스트림은 파이프라인의 중간 결과에 쉽게 접근할 수 없으므로, 중간 계산 결과를 여러 번 사용해야 하는 경우(예: 합계를 계산하고 그 합계를 기반으로 할인을 계산한 다음, 두 값을 모두 출력)에는 전통적인 방식이 더 적합합니다.

### 결론
> 스트림을 사용해야 할 때도 있고, 반복을 사용해하 할 때도 있다. 어느 쪽을 선택해야 하는지 확실한 기준은 없으니, 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라

### 부록
- 병렬 스트림
  
	> Java 스트림 API는 parallelStream()을 사용하여 병렬 처리(멀티스레딩)를 자동으로 수행할 수 있습니다.
 	```
	import java.util.List;

	public class ParallelStreamExample {
	    public static void main(String[] args) {
		List<String> list = List.of("apple", "banana", "cherry", "avocado", "blueberry");
	
		list.parallelStream()
		    .map(String::toUpperCase) // 대문자로 변환
		    .forEach(s -> System.out.println(Thread.currentThread().getName() + " - " + s));
	    }
	}
  	```
	```
 	// 실행결과 - forEach()는 순서를 보장하지 않음 → 실행 결과가 매번 다를 수 있음
	ForkJoinPool.commonPool-worker-4 - AVOCADO
	main - CHERRY
	ForkJoinPool.commonPool-worker-1 - BANANA
	ForkJoinPool.commonPool-worker-3 - BLUEBERRY
	ForkJoinPool.commonPool-worker-2 - APPLE
	```
