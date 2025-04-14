# 지역변수의 범위를 최소화하라
## 지역변수 범위를 최소화하는 이유
- 가독성 향상
- 오류 방지
- 유지보수 용이성
## 최소화하는 방법
1. 선언 시점
   - 변수는 처음 사용될 때 선언할 것
   - 선언과 동시에 초기화할 것
   - 초기화에 필요한 정보가 부족하면 충분해질 때까지 선언 미루기
2. try-catch문 처리
   - 변수를 초기화하는 표현식에서 예외 발생 가능성이 있다면 try 블록 안에서 초기화
   - 변수를 try 블록 밖에서도 사용해야 한다면 try 블록 앞에서 선언
    ```java
    // 변수를 try 블록 밖에서 선언하고 기본값으로 초기화
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader("data.txt"));
    } catch (IOException e) {
        System.err.println("파일을 열 수 없습니다: " + e.getMessage());
    } finally {
        // try 블록 밖에서도 br 변수에 접근해야 함
        if (br != null) {
            try {
                br.close();
            } catch (IOException e) {
                System.err.println("파일을 닫는 중 오류 발생: " + e.getMessage());
            }
        }
    }
    ```
3. 반복문 활용
    - for문이나 for-each문은 변수 범위를 반복문 내부로 제한
    - 반복 변수 범위를 명확히 제한하기 위해 while문보다는 for문 사용 권장
    - 컬렉션 순회 -> for(Element e : c)
    - 반복자가 필요할 때 -> for (Iterator i = c.iterator(); i.hasNext(); ) { ... }
    - 복사-붙여넣기 할 때도 유용
      ```java
      Iterator<Element> i = c.iterator();
      while (i.hasNext()) {
      }
      Iterator<Element> i2 = c2.iterator();
      while (i.hasNext()) { // i2가 아닌 i를 사용 → 실수 (에러 발생 X)
      }
      ---------------------------------------------------------
      for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
        Element e = i.next();
      }
      for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) { 
        // i를 찾을 수 없다는 컴파일 오류 발생
        Element e2 = i2.next();
      }
      ```
    - for문의 장점
      ```java
      for (int i = 0, n = expensiveComputation(); i < n; i++) {
          // n은 반복마다 다시 계산 안 함
      }
      ```
4. 메서드 설계
    - 메서드를 작게 유지하고 한 가지 기능에 집중
    - 여러 기능을 처리하는 메서드는 기능별로 쪼개기

## 정리
- 지역변수는 필요할 때만 선언하고, 사용 범위를 좁게 유지하자.
- 안전하고 유지보수 쉬운 코드 작성 가능!
