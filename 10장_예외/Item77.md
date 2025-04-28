# 예외를 무시하지 말라
## 예외를 무시하지 말자
- try문으로 감싼 후 catch 블록에서 아무 일도 하지 않는 습관은 좋지 않다.
- 예외를 무시하는 것은 화재경보를 꺼버려, 아무도 알지 못하게 하는 것과 같음.
## 예외를 무시해도 되는 경우
- FileInputStream을 닫을 때 -> 읽기 전용이므로 파일 상태가 변경되지 않으므로 복구할 것도 없음.
- 예외가 자주 발생한다면 로그로 남겨 확인하자.
- 예외를 무시하기로 정한 경우, 결정 이유와 예외 변수 이름도 ignored로 바꾸자.
```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numCoIors = 4; // 기본값. 어떤 지도라도 이 값이면 충분하다.
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) { 
    // 기본값올 사용한다(색상 수룰 최소화하면 좋지만, 필수는 아니다).
}
```
## 정리
- 검사 예외와 비검사 예외 모두에 적용되는 원칙
- 예외를 적절히 처리하거나 최소한 전파되도록 해야 함
- 그래야 오류를 방지하거나 최소한 디버깅 정보를 남기고 프로그램이 빠르게 중단되게 할 수 있음
