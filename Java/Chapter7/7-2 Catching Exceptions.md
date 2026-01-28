예외를 던졌으면 예외를 처리해야 한다. (Catching)
예외를 직접 처리할지 아니면 호출자에게 떠넘길지 결정해야한다.
## Catching and Exception

프로그램 어디에서도 예외를 잡지 않는다면?
- 콘솔 프로그램: 프로그램이 즉시 종료되며, 예외의 종류와 Stack trace(스택 추적) 메시지를 콘솔에 출력
- GUI 프로그램: 일반적으로 예외를 잡아 스택 추적을 출력한 뒤, 프로그램이 죽지 않고 사용자 인터페이스 처리 루프로 돌아간다.

try/catch 
: 예외를 잡기위해서는 try/catch 블록을 사용해야 한다.
```java
try
{
   // 1. 예외가 발생할 가능성이 있는 코드
   code
   more code
}
catch (ExceptionType e)
{
   // 2. 해당 예외 타입(ExceptionType)이 발생했을 때 실행할 처리 코드
   handler for this type
}
```

Execution Flow
- 1. 예외가 발생할 경우, try 블록 내의 코드를 실행하다가 catch 절에 명시된 예외가 발생하면
	- try 블록의 나머지 코드는 건너뛰고 catch 블록 안의 처리코드를 실행한다. 
	- 이후 try/catch 블록 다음의 코드를 실행한다.
- 2. 예외 발생 x, try 블록 실행하고 catch 절은 건너뛴다.
- 3. 다른 예외발생 시에, catch 절에 명시된 타입이 아닌 다른 예외가 발생하면, 이 메서드 종료


예외 처리의 전략
- 1. Catching : 오류가 발생하면 그 자리에서 로그를 남기거나 대응
```java
public void read(String filename)
{
   try
   {
      var in = new FileInputStream(filename);
      int b;
      while ((b = in.read()) != -1) // 여기서 IOException 발생 가능
      {
         // 입력 처리
      }
   }
   catch (IOException exception)
   {
      // 예외 발생 시 스택 추적 출력
      exception.printStackTrace();
   }
}
```
- 2. Propagating : 현재 메서드에서 예외를 처리할 방법이 없음 -> 예외를 호출자에게 넘기기
```java
// try-catch 블록이 사라지고, throws가 추가됨
public void read(String filename) throws IOException
{
   var in = new FileInputStream(filename);
   int b;
   while ((b = in.read()) != -1)
   {
      // 입력 처리
   }
}
```


예외 상황
- 슈퍼클래스의 메서드가 예외를 던지지 않는다면, 이를 오버라이딩 하는 서브클래스의 메서드도 Checked Exception을 던질 수 없다. 
- 따라서 그런 메서드를 오버라이딩 할 때는 코드 내에서 Checked Exception이 발생한다면, 반드시내부에서 try/catch로 처리해야한다. 밖으로 던지는 것이 금지되어 있기 때문에. 



## Catching Multiple Exceptions

예외별로 다르게 처리하기 (Separate Catch Blocks)
- 여러 유형의 예외 발생 가능 -> 대처 방법이 달라야 한다면
```java
try
{
   // 예외가 발생할 수 있는 코드
}
catch (FileNotFoundException e)
{
   // 파일이 없을 때의 긴급 조치
}
catch (UnknownHostException e)
{
   // 호스트를 찾을 수 없을 때의 긴급 조치
}
catch (IOException e)
{
   // 그 외 나머지 모든 입출력 문제에 대한 조치
}
```
- Implicit Rule : 자식 예외를 먼저 잡아야 한다. 
	- `FileNotFoundException` 자식, `IOException` 부모
	- 만약 `IOException`을 먼저 적으면, 모든 입출력 예외가 그곳에서 잡혀버려서 뒤에 있는 `FileNotFoundException` 블록은 영원히 실행되지 못한다. (Unreachable code)
- `catch` 블록의 변수 `e`를 통해 예외에 대한 상세 정보를 얻을 수 있다.
	- `e.getMessage()` : 상세 에러 메시지를 반환
	- `e.getClass().getName()` : 예외 객체의 실제 클래스 이름(타입)을 반환한다.

여러 예외를 한번에
- 서로 다른 예외지만 처리하는 동작이 같을경우
```java
try
{
   // 예외가 발생할 수 있는 코드
}
catch (FileNotFoundException | UnknownHostException e)
{
   // 파일이 없거나 호스트를 모를 때, 똑같은 조치를 취함
}
catch (IOException e)
{
   // 나머지 입출력 문제 처리
}
```
- 서로 상속 관계가 없는 예외들을 묶을 때 유용하다
	- 만약 상속 관계라면 그냥 부모만 적으면 되기 때문

멀티 캐치는 효율적이다.
- 컴파일된 바이트코드를 보면, 예외 처리 코드를 중복 생성하지 않아 프로그램의 크기가 줄어들고 효율적이다.


## Rethrowing and Chaining Exceptions

Exception Translation
: 서브시스템을 개발할 때, 내부 구현의 구체적인 오류를 외부 사용자에게 그대로 노출하는 것은 좋지 않을 수 있다. 대신 해당 서브시스템의 실패를 의미하는 더 추상적인 예외로 변환해서 던지는 것이 합리적이다.
```java
try {
    // 데이터베이스 접근
} catch (SQLException e) {
    // 구체적인 SQL 오류를 일반적인 서블릿 오류로 변환
    throw new ServletException("database error: " + e.getMessage());
}
```
- 새로운 예외를 생성하고 기존 예외의 메시지를 포함

Exception Chaining / Wrapping
: 위의 방법은 원본 예외의 유형과 상세 stack trace를 잃어버릴 수 있다는 단점이 있다. 이를 해결하기 위해 예외 연결을 사용하는 것이 권장된다.
```java
try {
    // 데이터베이스 접근
} catch (SQLException original) {
    var e = new ServletException("database error");
    e.initCause(original); // 원본 예외를 '원인'으로 등록
    throw e;
}
```
- `initCause()` --> 원본 예외를 새로운 예외의 원인으로 설정
- 사용자는 `ServletException` 이라는 상위 레벨의 오류를 받는다.
- 디버깅 시 `caughtException.getCause()` 호출을 통해 원본 `SQLException`을 볼 수 있다. --> 정보 손실 x
- **Checked 예외를 Runtime 예외로 포장**
	- 만약 오버라이딩 하려는 메서드가 Checked Exception을 던지는 것을 허용하지 않을때, 내부에서 Checked Exception이 발생한다면 --> `RuntimeException`으로 Wrap해서 던지면 된다.

Rethrowing without Change
: 때로는 예외를 처리하지 않고, 단순히 로그만 남기고 싶을 때가 있다. 이 경우 예외를 잡아서 로그를 찍고 그대로 다시 던진다.
```java
try {
    access the database
} catch (Exception e) {
    logger.log(level, message, e); // 기록
    throw e; // 다시 던지기
}
```



## The finally Clause

`finally`?
: 메서드 실행 중 예외가 발생하면 남은 코드를 실행하지 않고 즉시 메서드를 빠져나간다. 만약 해당 메서드가 파일이나 네트워크 연결 같은 자원을 가지고 있었다면, 예외 발생 시 해제 코드가 실행되지 않아 Resource Leak 가 발생할 수 있다.
이때 `finally` 블록을 사용한다. 이 블록 안의 코드는 예외 발생 여부와 상관없이 무조건 실행된다.

ex)
```java
var in = new FileInputStream(. . .);
try
{
   // 1
   code that might throw exceptions
   // 2
}
catch (IOException e)
{
   // 3
   show error message
   // 4
}
finally
{
   // 5
   in.close(); // 자원 해제
}
// 6
```
- 예외가 발생하지 않으면 1->2->5->6 으로 실행된다
- 예외가 발생하고 catch에서 잡았을 때, 1->3->4->5-> 이 된다
- 예외가 발생했으나 catch에서 못잡았을 때, 1->5->메서드 종료 및 예외 전파.
	- 예외가 호출자에게 다시 던져짐.


catch 없는 finally
: finally 절은 catch 절 없이도 사용할 수 있다. 예외 처리는 호출자에게 맡기고 여기서는 자원만 정리하겠다는 의도
```java
InputStream in = ...;
try
{
   code that might throw exceptions
}
finally
{
   in.close();
}
```

Decoupling
: 자원 해제와 오류 보고의 책임을 분리하기 위해 try 블록을 중첩해서 사용하는 것이 더 명확하고 기능적일 때가 있다. 
```java
try // 외부 블록: 오류 보고 책임
{
   try // 내부 블록: 자원 해제 책임
   {
      code that might throw exceptions
   }
   finally
   {
      in.close(); // 여기서 에러가 나도 외부 catch가 잡음
   }
}
catch (IOException e)
{
   show error message
}
```
- 장점은 `finally` 블록에서 `in.close()`를 하다가 발생한 오류까지도 외부 `catch` 블록에서 처리할 수 있다는 것이다.

주의
: `finally` 블록 안에는 return, throw, break, continue 등의 flow control을 하지 말아야 한다.
ex)
```java
public static int parseInt(String s)
{
   try
   {
      return Integer.parseInt(s);
   }
   finally
   {
      return 0; // 위험!!
   }
}
```
- 만약 `parseInt("42")` 가 호출됐을 때, try는 42를 반환하려고 하지만 finally가 실행되면서 0으로 바뀐다.
- Swallowing Exceptions : `parseInt("zero")`를 호출하면 `NumberFormatException`이 발생한다. 하지만 `finally` 블록의 `return 0;`이 실행되면서 예외가 사라지고 그냥 0을 반환한다.


## Try-with-resources

기존에는 자원을 사용한 후 `finally` 에서 일일이 `close()`를 호출해야 했다. try-with-resources 문을 사용하면, try 블록이 끝날 때 자동으로 자원의 `close()` 메서드가 호출된다. 

사용 조건
: 이 문법을 사용하려면 해당 자원 클래스가 `AutoCloseable` 인터페이스를 구현해야한다.
- `AutoCloseable` : 단 하나의 메서드 `void close() throws Exception`을 가진다.
- `closeable` : `AutoCloseable`의 하위 인터페이스로, 입출력 클래스들이 주로 구현한다. 
				`void close() throws IOException`을 가진다.

기본 문법
```java
// try 괄호 안에서 자원을 선언하고 초기화
try (var in = new Scanner(Path.of("in.txt"), StandardCharsets.UTF_8))
{
   while (in.hasNext())
      System.out.println(in.next());
} 
// 블록을 빠져나오면 자동으로 in.close()가 실행됨
```

Multiple Resources
: 여러 개의 자원을 동시에 관리할 땐 세미콜론으로 구분해서 나열
```java
try (var in = new Scanner(Path.of("in.txt"), StandardCharsets.UTF_8);
     var out = new PrintWriter("out.txt", StandardCharsets.UTF_8))
{
   while (in.hasNext())
      out.println(in.next().toUpperCase());
}
// 블록 종료 시 out.close()와 in.close()가 모두 호출됨
```

Java 9 개선점
: Java 9 이전에는 반드시 try 괄호 안에서 변수를 선언해야 했다. 하지만 Java 9부터는 이미 선언된 변수가 effectively final 이라면 그대로 괄호안에 넣을 수 있다.
```java
public static void printAll(String[] lines, PrintWriter out)
{
   try (out) // 기존 변수 out을 그대로 사용
   {
      for (String line : lines)
         out.println(line);
   } 
   // 여기서 out.close()가 호출됨
}
```


Suppressed Exceptions
: try-with-resources는 `try` 블록 내부 코드와 `close()` 메서드 양쪽에서 모두 예외가 발생하는 상황을 수월하게 처리한다.
- 기존 `finally` 방식 : 나중에 발생한 `close()` 예외가 처음 발생한 try 블록 내부의 예외를 덮어쓴다. (Swallowing)
- Try-with-resources : Original Exception(try 블록 내부의 예외)을 사용자에게 던진다.(Rethrow)
	- `close()` 에서 발생한 예외는 suppressed exception 으로 간주되어 Original Exception 에 덧붙여진다.
	- 필요하다면 `e.getSuppressed()` 메서드를 호출하여 억제된 예외들의 목록을 배열로 확인할 수 있다.

Try-with-resources 구문에도 `catch` , `finally`를 붙일 수 있다.
- try 블록 -> 자원 자동 종료(`close()`) -> catch 절 (예외 발생 시) -> finally 절 실행
- 즉, `catch` `finally` 실행 전에 이미 자원은 안전하게 회수됨



## Analyzing Stack Trace Elements

Stack Trace
: 프로그램 실행 중 특정 시점에 호출되어 아직 종료되지 않은 메서드들의 목록. 예외가 발행하고 프로그램이 종료될 때 콘솔에 출력되는 메시지들임

스택 추적에 접근하는 법
- `Throwable`
	- `printStackTrace()` 메서드를 사용하여 콘솔이나 문자열로 덤프를 뜬다. (Dump : 현재 상태를 그대로 복사해서 쏟아내는 것)
	- `getStackTrace()`를 호출하면 `StackTraceElement[]` 배열을 얻는다.
	- 단점 : 호출 시점에 전체 스택을 캡처한다. 스택이 깊은 경우성능상 비효율적이다. 또한, 클래스 이름만 알 수 있고 실제 객체에는 접근할 수 없다.
- `StackWalker` (Java 9+)
	- `StackWalker.StackFrame` 인스턴스의 스트림을 제공한다.
	- `walk()` 메서드를 사용하면 스트림을 통해 필요한 만큼만 스택 프레임을 처리할 수 있어서 효율적이다. (Lazy Evaluation)
	- 파일명, 라인 번호, 클래스 객체, 메서드 이름 등을 확인할 수 있다.


| **클래스**                  | **메서드**             | **설명**                                 |
| ------------------------ | ------------------- | -------------------------------------- |
| **StackWalker** (Java 9) | `getInstance()`     | `StackWalker` 인스턴스를 얻습니다.              |
|                          | `forEach(Consumer)` | 각 스택 프레임에 대해 작업을 수행합니다.                |
|                          | `walk(Function)`    | 스택 프레임 스트림을 처리하고 결과를 반환합니다. (지연 처리 가능) |
| **StackFrame** (Java 9)  | `getFileName()`     | 소스 파일 이름을 반환합니다.                       |
|                          | `getLineNumber()`   | 실행 중인 라인 번호를 반환합니다.                    |
|                          | `getClassName()`    | 클래스의 전체 이름(FQCN)을 반환합니다.               |
|                          | `getMethodName()`   | 메서드 이름을 반환합니다.                         |
| **Throwable**            | `printStackTrace()` | 스택 추적을 표준 오류 스트림에 출력합니다.               |
|                          | `getStackTrace()`   | `StackTraceElement` 배열을 반환합니다. (구형 방식) |















