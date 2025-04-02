# 가변인수는 신중히 사용하라

## 가변인수란
- 인자의 개수가 정해져 있지 않은 메서드 인자
- 명시한 타입의 인수를 0개 이상 받을 수 있음

```java
static int sum(int... args) {
  int sum = 0;
  for (int arg : args)
    sum += arg;
  return sum;
}

// 인수가 0개여도 문제 없음
sum(1, 2, 3) = 6;
sum() = 0;
```

## 인수가 1개 이상이어야 하는 가변인수 메서드
- 잘못된 코드
  - 인수를 0개만 넣을 경우 런타임에 실패할 수 있음
  - 유효성 검사를 명시적으로 해야 함
  - for-each문 사용 불가
```java
static int min(int... args) {
  if (args.length == 0)
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
  int min = args[0];
  for (int i = 1; i < args.length; i++)
    if (args[i] < min)
      min = args[i];
  return min;
```

- 매개변수를 2개 받도록 하여 해결 가능
  - 첫 번째로 평범한 매개변수, 두 번째로 가변인수
```java
static int min(int firstArg, int... remainingArgs) {
  int min = firstArg;
  for (int arg : remainingArgs)
    if (arg < min)
      min = arg;
return min;
}
```

## 가변인수가 걸림돌이 되는 상황
- 호출될 때마다 배열을 새로 하나 할당하고 초기화함
- 이 문제를 해결하기 위해 특수한 상황을 고려한 코드를 짤 수 있음
  - 인수가 0개인 것부터 4개인 것까지 5개를 다중정의할 것.
  - EnumSet의 정적팩터리도 같은 방식
  - 마지막 다중정의 메서드가 인수 4개 이상인 5% 호출을 담당
