# 추상화 수준에 맞는 예외를 던지라

## 문제점: 추상화 수준이 다른 예외를 그대로 던질 때

- 저수준 예외를 상위 계층에 그대로 던지면 내부 구현이 노출됨
- API 사용자는 해당 구현 방식에 의존하게 됨
- 내부 구현을 바꾸면 다른 예외가 발생하고, 기존 코드에 영향을 줌
- 추상화 계층 간의 경계를 흐리게 만들어 유지보수가 어려워짐


## 해결책 1: 예외 번역 (Exception Translation)

- 저수준 예외를 포착해, 자신의 추상화 수준에 맞는 예외로 변환
- 고수준 API는 내부 구현에 의존하지 않게 됨

```java
try {
  ... // 저수준 API 호출
} catch (SQLException e) {
  throw new DataAccessException("데이터 조회 실패", e);
}
```

## 해결책 2: 예외 연쇄 (Exception Chaining)

- 고수준 예외를 던질 때, 원인 예외를 함께 포함시켜 전달
- 디버깅 및 로깅 시 유용

```java
try {
  ... // 파일 읽기
} catch (IOException e) {
  throw new FileProcessingException("파일 처리 중 오류 발생", e);
}
```

## 💡 결론

- 예외는 추상화 수준에 맞게 던져야 한다.
- 고수준 API에서는 내부 구현을 드러내지 말고, 의미 있는 예외로 변환하자.
- 예외 번역과 연쇄를 통해 유지보수가 쉬운 코드 구조를 만들 수 있다.

---
## 부록

### 저수준 예외 (Low-level Exception)

- 시스템 API, 라이브러리, DB 등에서 발생하는 예외
- 내부 세부 구현과 밀접하게 연관
- 클라이언트에게 노출할 필요가 없는 예외
- `IOException`, `SQLException`, `FileNotFoundException`, `SocketException`

```java
try {
  Connection conn = DriverManager.getConnection(...); // SQLException 발생 가능
} catch (SQLException e) {
  e.printStackTrace();
}
```

### 고수준 예외 (High-level Exception)

- 비즈니스 또는 도메인 로직 관점에서 발생하는 예외
- 클라이언트에게 **의미 있는 메시지**를 전달
- 저수준 예외를 감싸거나 변환하여 처리
- `UserNotFoundException`, `PaymentFailureException`, `DataAccessException`

```java
try {
  userRepository.findById(userId);
} catch (SQLException e) {
  throw new UserRetrievalException("사용자 조회에 실패했습니다", e);
}
```

### 요약 비교

| 항목 | 저수준 예외 | 고수준 예외 |
|------|-------------|-------------|
| 발생 위치 | 시스템, DB, 라이브러리 내부 | 비즈니스 로직, 도메인 계층 |
| 사용자 노출 | 내부 세부사항 노출됨 | 의미 중심으로 설명됨 |
| 대표 예외 | `SQLException`, `IOException` | `UserNotFoundException`, `DataAccessException` |
| 처리 방식 | 상위 계층에서 포장 또는 변환 필요 | 클라이언트가 직접 처리 가능 |
