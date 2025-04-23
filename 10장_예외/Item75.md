# 예외의 상세 메시지에 실패 관련 정보를 담으라

>  예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적(stack trace) 정보를 자동으로 출력한다. <br>
스택 추적은 예외 객체의 toString 메서드를 호출해 얻는 문자열 <br>
예외의 클래스 이름 뒤에 상세 메시지가 붙는 형태

→ 실패 원인을 분석하기위해, 예외의 toString 메서드에 실패원인에 관한 정보를 가능한 한 많이 담아 반환해야한다.


- 실제

```java
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3
Exception in thread "main" java.lang.IndexOutOfBoundsException: Index 1 out of bounds for length 0
```

- 권장

```
Exception in thread "main" java.lang.IndexOutOfBoundsException: 최솟값: 0, 최댓값: 10, 인덱스: 15
```
```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index){
        // 상세한 오류 메시지를 생성
        super(String.format(
                "최솟값: %d, 최댓값: %d, 인덱스: %d",
                lowerBound, upperBound, index));
                
        this.lowerBound = lowderBound;
        this.upperBound = upperBound;
        this.index = index;
}
```

### 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.

> 예를들어 IndexOutOfBoundsException의 상세 메시지는 범위의 최솟값과 최댓값, 그리고 그 범위를 벗어났다는 인덱스의 값을 담아야한다. <br>
관련 데이터를 모두 담아야 하지만 장황할 필요는 없다.


### 예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안된다.

> 최종 사용자에게는 친절한 안내 메시지를 보여줘야하지만, 예외 메시지의 주 소비층은 문제를 분석해고 해결하는 개발자이다. 따라서 가독성보다는 안에 담긴 내용들이 훨씬 중요하다.


### 실패를 적절히 포착하려면 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 괜찮다.

> 예를 들어 현재의 IndexOutOfBoundsException 생성자는 String을 받지만, `권장` 코드와 구현했어도 좋다. <br>
자바 9에서는 IndexOutOfBoundsException에 정수 인덱스 값을 받는 생성자가 추가되었다. 하지만 아쉽게도 최솟값과 최댓값까지 받지는 않는다.
      

### 예외는 실패와 관련된 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는 것이 좋다.

> 포착한 실패 정보는 예외 상황을 복구하는 데 유용할 수 있으므로 접근자 메서드는 비검사 예외보다는 검사 예외에서 더 빛을 발한다.


### 활용

```
/**
 * 카드 잔고가 부족할 때 발생하는 검사 예외
 */
public class InsufficientFundsException extends Exception {

    /**
     * 필요한 금액과 사용 가능한 금액을 받아 부족한 금액을 계산하는 생성자
     *
     * @param requiredAmount 결제에 필요한 금액
     * @param availableAmount 카드에서 사용 가능한 금액
     */
    public InsufficientFundsException(double requiredAmount, double availableAmount) {
        // 상세 메시지 생성
        super(String.format(
                "카드 잔고 부족: 필요한 금액 %.2f원, 사용 가능한 금액 %.2f원, 부족한 금액 %.2f원", 
                requiredAmount, availableAmount, (requiredAmount - availableAmount)));
        
        this.requiredAmount = requiredAmount;
        this.availableAmount = availableAmount;
        this.shortageAmount = requiredAmount - availableAmount;
    }
    
    /**
     * 결제에 필요한 총 금액을 반환하는 접근자 메서드
     */
    public double getRequiredAmount() {
        return requiredAmount;
    }
    
    /**
     * 카드에서 사용 가능한 금액을 반환하는 접근자 메서드
     */
    public double getAvailableAmount() {
        return availableAmount;
    }
    
    /**
     * 부족한 금액을 반환하는 접근자 메서드
     */
    public double getShortageAmount() {
        return shortageAmount;
    }
}
```

```
public class PaymentProcessor {
    
    /**
     * 카드로 결제를 처리하는 메서드
     * 
     * @param cardNumber 카드 번호
     * @param amount 결제 금액
     * @throws InsufficientFundsException 카드 잔고가 부족한 경우
     */
    public void processPayment(String cardNumber, double amount) throws InsufficientFundsException {
        // 카드 잔고 확인 (실제로는 카드사 API 등을 통해 확인)
        double cardBalance = getCardBalance(cardNumber);
        
        // 잔고가 부족한 경우 예외 발생
        if (cardBalance < amount) {
            throw new InsufficientFundsException(amount, cardBalance);
        }
        
        // 결제 처리 코드...
        System.out.println("결제가 완료되었습니다. 금액: " + amount + "원");
    }
    
    // 카드 잔고를 확인하는 메서드 (예시)
    private double getCardBalance(String cardNumber) {
        // 실제로는 카드사 API 등을 통해 잔고를 확인
        // 여기서는 예시로 간단하게 구현
        return 5000.0; // 예시: 5,000원의 잔고
    }
    
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        
        try {
            // 10,000원 결제 시도 (잔고는 5,000원)
            processor.processPayment("1234-5678-9012-3456", 10000.0);
        } catch (InsufficientFundsException e) {
            System.err.println(e.getMessage());
            
            // 접근자 메서드를 통해 상세 정보에 접근
            System.err.println("결제를 위해 추가로 필요한 금액: " + e.getShortageAmount() + "원");
            
            // 추가 처리 (예: 다른 결제 수단 제안)
            System.out.println("다른 결제 수단을 이용하시겠습니까?");
        }
    }
}
```