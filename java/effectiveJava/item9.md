# 아이템 9 try-finally 보다는 try-with-resources를 사용하라

- 자바 라이브러리에서 close 메서드를 호출해 직접 닫아줘야 하는 자원
    - InputStream, OutputStream, java.sql.Connection 등..
    - 자원 닫기는 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제로 이어지기도 한다


### try-finally

- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 쓰임
- 예외가 발생하거나 메서드에서 반환되는 경우 포함

```java
// try-finally 더 이상 자원을 회수하는 최선의 방책이 아니다
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

### 자원을 하나 더 사용한다면..?

```java
// 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        } finally{
            in.close();
        }
    }
}
```

- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 예컨대 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패할 것이다
- 이런 상황에서 두번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다
- 그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어 실제 시스템에서의 디버깅을 몹시 어렵게 한다

### try-with-resources 로 해결하자!

```java
// 첫번째 예제 코드를 try-with-resources로 재작성한 예
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        } finally{
            in.close();
        }
    }
}
```

```java
// 두번째 에제 코드를 try-with-resources로 재작성한 예
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

- 읽기 수월
- 문제 진단하기도 훨씬 좋음

```java
// try-with-resources를 catch 절과 함께 쓰는 모습
static String firstLineOfFile(String path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(
				new FileReader(path))) {
			return br.readLine();
	} catch (IOException e) {
			return defaultVal;
	}
}
```

- catch 절 덕분에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다
- firstLineOfFile 메서드를 살짝 수정해 파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환하도록 한 예시

### 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자

- 예외는 없다.
- 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다
- try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도 try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다