# 라이브러리를 익히고 사용하라

## 라이브러리를 적절히 사용하지 않은 예
- 무작위 정수 하나를 생성한다고 가정해보자.

    ```java
    static Random rnd = new Random();

    static int random(int n) {
        return Math.abs(rnd.nextInt()) % n;
    }
    ```

- 3가지 문제점

    1. n이 그리 크지 않은 2의 제곱수라면 수열이 반복된다.

    2. n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다.<br>
    (n값이 크면 이 현상은 더 두드러진다.)
        ```java
        public static void main(String[] args) {
            int n = 2 * (Integer.MAX_VALUE / 3);
            int low = 0;
            for (int i = 0; i < 1_000_000; i++)
                if (random(n) < n/2)
                    low++;
            System.out.println(low);
        }
        ```

        > 실제 코드를 실행시켰을 때 다음과 같은 결과가 나왔다.<br>
        > 666629, 667442, 667206

    3. 지정함 범위 '바깥'의 수가 종종 튀어나올 수 있다.
        - rnd.nextInt()가 반환한 값을 Math.abs를 이용해 음수가 아닌 정수로 매핑하기 때문이다.
        - Math.abs(Integer.MIN_VALUE) == Integer.MIN_VALUE

## 라이브러리를 적절히 사용한 예

- 위 문제를 해결하기 위해서는 이미 문제들이 해결되어 있는 `Random.nextInt(int)`를 사용하면 된다.
  - 자바 7부터는 ThreadLocalRandom을 사용하는 것이 더 좋다.
  - 포크-조인 풀이나 병렬 스트림에서는 SplittableRandom을 사용하는 것이 좋다.

- 표준 라이브러리를 사용하는 5가지 이점

    1. 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.

    2. 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다.

    3. 따로 노력하지 않아도 성능이 지속해서 개선된다.

    4. 기능이 점점 많아진다.

    5. 내가 작성한 코드가 많은 사람에게 낯익은 코드가 된다.

## 결론

- 자바 프로그래머라면 적어도 java.lang, java.util, java.io와 그 하위 패키지들에는 익숙해지자.

- 그 중에서도 특히 컬렉션 프레임워크와 스트림 라이브러리, java.util.concurrent의 동시성 기능은 알아두면 큰 도움이 된다.
