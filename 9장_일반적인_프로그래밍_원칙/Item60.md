#  정확한 답이 필요하다면 float와 double은 피하라

> float와 double 타입은 과학과 공학 계산용으로 설계되었다. 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 ‘근사치’로 계산하도록 세심하게 설계되었다. 따라서 정확한 결과가 필요할 때는 사용하면 안 된다.

### float

> 1개의 부호비트와 8개의 지수비트 23개의 가수비트로 이루어져 있다.

### double

> 1개의 부호비트와 11개의 지수비트 52개의 가수비트로 이루어져 있다.

```java
System.out.println(1.03 - 0.42); // 0.6100000000000001 != 0.61
```

## 극복

- BigDecimal 사용
```java
import java.math.BigDecimal;

public class BigDecimalArithmetic {
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("100.25");
        BigDecimal num2 = new BigDecimal("99.75");

        // 덧셈
        BigDecimal sum = num1.add(num2);
        System.out.println("합: " + sum);  // 200.00
        }
    }
}
```
> 기본 타입보다 쓰기가 훨씬 불편하고, 훨씬 느리다.

- int, long으로 변환
> 10의 거듭제곱을 곱해서 정수로 변환해놓고 사용한다. <br>
그럴 경우 다룰수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.

## 결론
> 숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟 자리 십진수로 표현할 수 있다면 long을 사용하라. 열여덟 자리를 넘어가면 BigDecimal을 사용해야 한다

# 부록

### 수 저장 방식

> $$(가수부분)\times2^{(지수부분)}$$ 형태로 수를 저장하는 방법. <br>
지수부분은 -127의 bias를 갖고, 가수부분은 항상 1로 시작한다. <br>
최댓값은 <br>
![](https://velog.velcdn.com/images/kimsz123/post/ccb4006d-bd5b-4a2e-bad2-0fa1df4c470a/image.png) <br>
최솟값은 (MIN_VALUE 이지만 절대적인 최솟값이 아니라 가장 작은 표현 단위) <br>
![](https://velog.velcdn.com/images/kimsz123/post/be78b576-0994-4d43-86ee-755032f7250c/image.png) <br>



※ 예외사항
> 지수부분은 00000001부터 11111110까지만 사용한다 (-126~127)
- 255는 INF와 NaN을 표현한다.
11111111에서 가수부분이 0이면 INF, 0이아니면 NaN
- 0은 0을 정교하게 표현하기 위해 사용한다.
기존의 가수부는 1로 시작하기 때문에 0표현이 불가능한데, 이를 위해 00000000의 경우에는 가수부분이 0부터 시작한다.

### 이진수 변환 방식

![](https://velog.velcdn.com/images/kimsz123/post/f714eba4-62fe-4c6f-bbd8-6cb2b5f0b2d3/image.png) <br>

0.1을 이진수로 표현하면 다음과 같이 반복되는 이진수가 됩니다:

![](https://velog.velcdn.com/images/kimsz123/post/75ea0510-c725-43f4-b83b-4ac99a44fa4c/image.png)

즉, 0.1을 이진수로 정확하게 표현하는 것은 불가능하며, 항상 일정한 비율로 반복되는 값이 됩니다.

따라서 위와 같은 이유로 double의 0.1과 0.2를 더하면 0.3이 아닌값이 된다.

![](https://velog.velcdn.com/images/kimsz123/post/9a71dd64-04a1-446c-b152-8ead2e0b5972/image.png)

![](https://velog.velcdn.com/images/kimsz123/post/47015c89-bac7-45bd-8488-43992cc05770/image.png)