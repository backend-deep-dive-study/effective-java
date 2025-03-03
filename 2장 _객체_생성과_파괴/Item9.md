# try-finally보다는 try-with-resources를 사용하라

## try-finally

- 자바 라이브러리에는 `close()`를 통해 직접 닫아줘야 하는 자원이 많이 존재한다.
   - InputStream, OutputStream, java.sql.Connection 등

   - 이에 대한 안전망으로 finalizer를 활용하지만 믿음직하지 못하다.

- 따라서 자원의 닫힘을 보장하기 위해 `try-finally`를 전통적으로 사용했다.

### 단점 1. 코드 가독성 저하

```java
Static void copy(String src, String dst) throws lOException { 
   InputStream in = new FileInputStreamfsrc);
   try {
      OutputStream out = new FileOutputStream(dst);
      try {
         byte[] buf = new byte[BUFFER_SIZE]： 
         int n;
         while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
      } finally {
         out.close();
      }
   } finally { 
      in.close();
   }
}
```

- `close()`를 호출해야 하는 자원이 2개만 되어도 코드가 복잡해진다.

### 단점 2. 디버깅의 어려움

```java
public static String readFirstLineFromFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine(); //1번
    } finally {
        br.close(); // 2번
    }
}
```

1. 위 코드에서 디스크에 물리적인 문제 등을 이유로 1번에서 `IOException`이 발생한다.

2. 코드가 finally로 이동하고, 같은 디스크 문제로 또 다른 `IOException`이 발생한다.

3. finally 블록의 예외가 try 블록의 예외를 덮어쓴다.

- 위 예시처럼 사용자는 파일을 닫는 과정의 오류만 보게 되어 디버깅이 어려워지게 된다.

### 단점 3. 예외 처리 및 로직 분리의 어려움

```java
public static void processFile(String path) {
    FileInputStream fileIn = null;
    try {
        fileIn = new FileInputStream(path);
        try {
            // 파일 데이터 읽기
            byte[] data = new byte[1024];
            int bytesRead = fileIn.read(data);
            
            // 데이터 처리
            processData(data, bytesRead);
            
        } catch (DataFormatException e) {
            // 데이터 형식 오류 처리
            System.err.println("데이터 형식 오류: " + e.getMessage());
        }
        // 다른 예외는 상위로 전파됨
    } catch (FileNotFoundException e) {
        // 파일 없음 오류 처리
        System.err.println("파일을 찾을 수 없음: " + e.getMessage());
    } catch (IOException e) {
        // 기타 IO 오류 처리
        System.err.println("IO 오류: " + e.getMessage());
    } finally {
        // 자원 정리
        if (fileIn != null) {
            try {
                fileIn.close();
            } catch (IOException e) {
                System.err.println("파일 닫기 오류: " + e.getMessage());
            }
        }
    }
}
```

- 중첩된 try-catch-finally 블록으로 인해 매우 복잡하다.

- 들여쓰기 수준이 깊어져 가독성이 떨어진다.

- 자원 관리와 비즈니스 로직, 예외 처리가 모두 섞여 있다.

## try-with-resources

- 자바 7부터 지원하는 `try-with-resources` 방식은 `try-finally`의 단점을 모두 해결해준다.

- 아래 코드는 `단점 1`의 코드를 `try-with-resources` 방식으로 작성한 코드이다.

   ```java
   static void copy(String src, String dst) throws lOE乂ception {
      try (
         InputStream in = new FilelnputStreani(src);
         Outputstream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) 
               out.write(buf, 0, n);
      }
   }
   ```

- 예외 처리 또한 향상되며, `단점 2`의 디버깅 문제도 해결했다.

   - try 블록과 자원 닫기 과정 모두에서 예외가 발생하면, try 블록의 원래 예외가 주 예외(primary exception)로 전파된다.

   - 자원을 닫는 과정에서 발생한 예외는 '숨겨진진 예외(suppressed exception)'로 주 예외에 첨부된다.

   ```
   java.io.IOException: 파일 읽기 실패 (readLine에서 발생한 예외)
      at ... (readLine의 스택 트레이스)
      at ... (메소드 호출 스택)
      Suppressed: java.io.IOException: 스트림 닫기 실패 (close에서 발생한 예외)
         at ... (close의 스택 트레이스)
   ```

- 마지막으로 여러 유형의 예외를 순차적으로 나열하여 처리할 수 있게 되며, `단점 3`의 문제도 해결했다.

   ```java
   public static void processFile(String path) {
      try (FileInputStream fileIn = new FileInputStream(path)) {
         // 파일 데이터 읽기
         byte[] data = new byte[1024];
         int bytesRead = fileIn.read(data);
         
         // 데이터 처리
         processData(data, bytesRead);
         
      } catch (FileNotFoundException e) {
         // 파일 없음 오류 처리
         System.err.println("파일을 찾을 수 없음: " + e.getMessage());
      } catch (DataFormatException e) {
         // 데이터 형식 오류 처리
         System.err.println("데이터 형식 오류: " + e.getMessage());
      } catch (IOException e) {
         // 기타 IO 오류 처리
         System.err.println("IO 오류: " + e.getMessage());
      }
      // 자원은 자동으로 닫힘
   }
   ```
