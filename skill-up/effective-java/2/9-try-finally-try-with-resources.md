# 아이템9 - try-finally보다는 try-with-resources를 사용하라

* 자바 라이브러리에는 `close()` 메서드를 호출해 직접 닫아줘야 하는 자원이 많다
  * InputStream, OutputStream, java.sql.Connection 등
* 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다

### try-finally

```java
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
        }
    } finally {
      	in.close();
    }
}
```

* 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다
* 또한 finally 내에서 발생한 예외로 인해 try 내에서 발생한 예외가 무시될 수 있다
* 이러한 문제는 `try-with-resources` 로 해결됨
  * 이 구조를 사용하려면 `AutoCloseable` 인터페이스를 구현해야 한다

### try-with-resources

```java
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

* 자원이 복수인 경우 try-with-resources 방식이 좋다
* 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다
* 숨겨진 예외들도 버려지지 않고 stacktrace에 (suppressed)를 달고 출력된다
