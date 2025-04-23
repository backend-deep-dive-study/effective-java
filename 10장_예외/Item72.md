# 표준 예외를 사용하라
## 표준 예외를 재사용하자
- API를 익히고 사용하기 쉬워짐
- 예외 클래스 수가 적을수록 메모리 사용량과 클래스 적재 시간이 줄어듦
## 재사용 가능한 표준 예외들
- IllegalArgumentException: 부적절한 인수 값을 전달했을 때 (ex: 반복 횟수에 음수 전달)
- IllegalStateException: 객체 상태가 메서드 수행에 적합하지 않을 때 (ex: 초기화되지 않는 객체를 사용할 때)
- NullPointException: null을 허용하지 않는 메서드에 null을 전달했을 때
- IndexOutOfBoundsException: 인덱스가 허용 범위를 벗어났을 때
- ConcurrentModificationException: 단일 스레드용 객체를 여러 스레드가 동시에 수정하려 할 때
- UnsupportedOperationException: 객체가 요청된 동작을 지원하지 않을 때 (ex: 원소를 넣을 수만 있는 List 구현체에 remove 메서드를 호출할 때)
## 주의사항
- Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말 것
  - 추상 클래스로 취급해야함.
  - 예외의 의미가 너무 포괄적.
  - 테스트 어려움.
- 예외의 이름뿐만 아니라 예외가 던져지는 맥락도 부합할 때만 재사용할 것
- 필요 시 표준 예외를 확장하여 더 많은 정보를 제공할 수 있음
- 어떤 예외를 선택할지 모호한 경우의 규칙: 인수 값이 무엇이든 실패했을 거라면 
IllegalStateException, 그렇지 않으면 IllegalArgumentException을 사용
