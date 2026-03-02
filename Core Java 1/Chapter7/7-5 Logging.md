
`System.out.println` 디버깅 방식
: 대부분의 Java 프로그래머는 문제가 생기면 `System.out.println` 을 곳곳에 심어서 변수값을 확인해야 한다.
- 문제는 원인을 찾고 나면 이 출력문들을 일일이 지워야 한다. 그런데 다른 문제가 생기면 다시 넣고 또 지우는 과정을 반복해야한다.
- 로깅 API를 이용하면 로그 코드를 지우지 않고도 출력 여부만 제어할 수 있다.

주요 장점
1. 모든 로그 기록을 끄거나, 특정 레벨 이하만 무시하는 것을 쉽게 할 수 있다.
2. 로그를 끄면, 해당 코드가 프로그램에 남아 있어도 성능 저하가 거의 없다
3. 로그를 콘솔에 출력할 수도 있고 파일에 저장할 수도 있고 네트워크로 보낼 수도 있다. 이를 처리하는 것이 핸들러이다.
4. 필터를 사용하여 작성자, 날짜 등의 다양한 기준으로 불필요한 로그를 걸러낼 수 있다.
5. 로그 기록을 일반 텍스트뿐만 아니라 XML 등 다양한 형식으로 저장할 수 있다
6. 패키지 이름처럼 계층적인 이름을 가진 여러 로거를 만들어 체계적으로 관리할 수 있다.
7. 코드를 수정하지 않고 외부 설정 파일만으로 로깅 동작을 제어할 수 있다.

## Basic Logging

복잡한 설정 없이 간단하게 로그를 남기고 싶다면 Global Logger (전역 로거)를 사용하면 된다
- 로그 남기는 방법
```java
//전역 로거를 호출하여 INFO 레벨의 로그를 기록
Logger.getGlobal().info("File->Open menu item selected");
```
- 출력 형식
```
May 10, 2013 10:12:15 PM LoggingImageViewer fileOpen
INFO: File->Open menu item selected
```
- 비활성화
```java
// 프로그램 시작 부분에서 레벨을 OFF로 설정하면 된다
Logger.getGlobal().setLevel(Level.OFF);
// 이후 모든 Logger.getGlobal() 호출은 무시된다.
```



## Advanced Logging

Defining Your Own Loggers
: 모든 로그를 하나의 전역 로거에 섞어서 관리하기 보다는 기능별, 패키지별로 로거를 따로 만든다.
- `Logger.getLogger` 메서드 사용
```java
private static final Logger myLogger = Logger.getLogger("com.mycompany.myapp");
```
- 생성한 로거를 아무 변수에도 저장하지 않으면 Garbage Collector 가 수거해 버릴 수 있다. 이를 방지하기 위해 위 예시처럼 `static`변수에 저장하여 참조를 유지하는 것이 좋다.
- 로거의 이름은 단순한 문자열이 아니라 부모-자식 관계를 형성한다. `"com.mycompany"` 는 `"com.mycompany.myapp"`의 부모이다. 자식로거는 부모 로거의 설정(로그레벨)을 상속받는다. 부모의 레벨을 변경하면 자식들에게도 적용된다.

Logging Levels
: Java 로깅 시스템은 로그의 중요도에 따라 7가지 레벨이 있다.

| **레벨 (높음 → 낮음)** | **설명**    | **기본 동작**      |
| ---------------- | --------- | -------------- |
| **SEVERE**       | 심각한 오류    | 기록됨            |
| **WARNING**      | 경고        | 기록됨            |
| **INFO**         | 일반 정보     | 기록됨 (기본값 커트라인) |
| **CONFIG**       | 설정 정보     | 무시됨            |
| **FINE**         | 상세 정보     | 무시됨            |
| **FINER**        | 더 상세한 정보  | 무시됨            |
| **FINEST**       | 가장 상세한 정보 | 무시됨            |
- 기본 설정: `INFO` 이상의 레벨만 기록된다. 

레벨 변경
``` java
// FINE 레벨 이상을 기록하도록 설정
logger.setLevel(Level.FINE);
```
- 로거의 레벨을 `FINE`으로 낮췄는데도 로그가 안 보일 수 있다. 이는 로그를 실제로 화면에 뿌려주는 핸들러가 여전히 `INFO` 이상만 출력하도록 설정되어 있기 때문이다.

로그 기록 메서드
: 각 레벨에 해당하는 메서드를 직접 호출하거나, `log` 메서드에 레벨을 인자로 넘길 수 있다.
```java
// 방법 1: 직관적인 메서드 사용
logger.warning("Something bad happened");
logger.fine("Detailed trace");

// 방법 2: log 메서드 사용
logger.log(Level.FINE, "Detailed trace");
```

Tracing Execution Flow
: 디버깅을 위해 메서드가 언제 시작하고 끝나는지 추적할 때 유용한 메서드들. 기본적으로 `FINER`레벨로 기록된다.
- `logp` : 클래스와 메서드 이름을 직접 명시
```java
logger.logp(Level.FINE, "MyClass", "myMethod", "Log message");
```
- `entering`, `exiting` : 메서드의 시작과 끝 기록, 매개변수나 반환값을 함께 기록
```java
int read(String file, String pattern) {
    // 진입 기록 (메서드 이름, 파라미터 배열)
    logger.entering("com.mycompany.Reader", "read", new Object[] { file, pattern 
    });
    
    // ... 작업 수행 ...
    
    // 종료 기록 (메서드 이름, 결과값)
    logger.exiting("com.mycompany.Reader", "read", count);
    return count;
}
// 로그에는 ENTRY, RETURN 과 같은 접두어가 붙어 출력된다.
```

Logging Exceptions
: 예외가 발생했을 때 이를 기록하는 두가지 주요 패턴이 있다.
- `throwing` : 예외를 throw 하기 직전에 기록을 남김 (`FINER` 레벨, 메시지는 `THROW`로 시작)
```java
if (errorCondition) {
    var e = new IOException("Error details");
    logger.throwing("MyClass", "read", e);
    throw e;
}
```
- `log` : catch 블록에서 예외를 처리할 때 스택 트레이스를 함께 남긴다.
```java
try {
    // ...
} catch (IOException e) {
    // 메시지와 함께 예외 객체(e)를 전달
    logger.log(Level.WARNING, "Reading image failed", e);
}
```


## Changing the Log Manager Configuration

설정파일만 수정하여 로깅 시스템의 동작(레벨, 출력방식)을 제어하는 방법.

설정 파일의 기본 위치
- Java 9 이후 : `jdk/conf/logging.properties`
- Java 9 이전 : `jre/lib/logging.properties`

기본 파일 대신 별도의 설정 파일을 사용하려면, 어플리케이션 실행 시 시스템 속성으로 경로를 지정해야 한다.
- `java -Djava.util.logging.config.file=myLogging.properties MainClass`

설정 파일(`logging.properties`)를 편집하여 전체 또는 특정 로거의 레벨을 조정할 수 있다.
- 기본(전역) 레벨 변경 : 모든 로거의 기본 레벨을 설정
	- `.level=INFO`
- 사용자 정의 로거 레벨 변경 : 특정 패키지나 클래스의 로거 레벨만 따로 설정하려면, 로거 이름 뒤에 `.level` 을 붙인다.
	- `com.mycompany.myapp.level=FINE`


주의
: 로거의 레벨을 `FINE`으로 낮췄는데 콘솔에 로그가 안 나온다면, 콘솔 핸들러의 레벨도 확인해야 한다.
- 로그는 로거(생산자) -> 핸들러(출력자) 순서로 전달된다. 따라서 설정 파일에 아래와 같이 추가해야 한다.
	- `java.util.logging.ConsoleHandler.level=FINE`

Runtime Initialization
: 명령줄 옵션 `-D` 를 사용하지 않고, 코드 내부에서 설정을 로드하고 싶을 때 사용
- 기본 초기화 (Java 8 이하)
	- `System.setProperty`로 파일 경로를 지정한 뒤, `readConfiguration()`을 호출하여 로그 매니저를 재초기화한다.
```java
System.setProperty("java.util.logging.config.file", "/path/to/myLogging.properties");
LogManager.getLogManager().readConfiguration();
```
- Java 9의 `updateConfiguration`
	- Java 9 이후 설정을 덮어쓰는 대신, 기존 설정과 병합하거나 세밀하게 업데이트하는 `updateConfiguration` 메서드가 도입됨
	- 파일에서 읽은 새 설정과 메모리에 있는 기존 설정을 어떻게 합칠지 결정하는 Mapper 함수를 제공해야한다.
		-  Mapper : `Function<String, BiFunction<String, String, String>>`
			- (Key) -> ((oldvalue, newvalue) -> resultvalue)
		- ex1) Merge preferring new : 키가 중복되면 새 파일의 값을 사용하고, 없으면 기존 값을 유지한다.
		```java
		LogManager.getLogManager().updateConfiguration(
			key -> ((oldValue, newValue) -> newValue == null ? oldValue : newValue)	
		);
		```
		- ex2) Partial Update : `com.mycompany`로 시작하는 설정만 새 값으로 바꾸고, 나머지는 기존 값을 유지
		```java
		LogManager.getLogManager().updateConfiguration(
		    key -> key.startsWith("com.mycompany")
	        ? ((oldValue, newValue) -> newValue)  // 해당 패키지는 새 값 적용
	        : ((oldValue, newValue) -> oldValue)  // 나머지는 기존 값 유지
		);
		```


주의사항
- 설정 파일 내부의 키 (ex: `com.mycompany.myapp.level`) 는 시스템 속성이 아니다. 따라서 `java -Dcom.mycompany.myapp.level=FINE...` 처럼 실행해도 효과가 없다.
- 실행 중인 프로그램의 로깅 레벨을 `JConsole` 도구를 통해 실시간으로 변경할 수 있다.
- `java.util.logging.manager` 속성을 설정하여 기본 `LogManager` 대신 다른 서브클래스를 사용하거나, 초기화 과정을 아예 우회할 수 있다. 



## Localization

이 섹션은 로그 메시지를 다양한 국가의 사용자가 읽을 수 있도록 국제화하는 방법을 다룬다.

Resource Bundles
: 이는 지역별로 다른 텍스트 매핑 정보를 담고 있는 저장소이다.
- key, value 쌍으로 이루어져 있다.
- 예시: `readingFile`이라는 키에 대해
	- 미국(en): `"Reading file"`
	- 독일(de): `"Achtung! Datei wird eingelesen"`
: 작성방법
- 각 언어별로 `.properties` 파일을 작성하여 클래스 파일과 같은 경로에 저장한다. 파일 이름에는 언어코드 (`en`, `de` 등)가 포함되어야 한다.
- 파일 이름 예시 : `com/mycompany/logmessages_de.properties`
- 파일 내용 예시 :
```
readingFile=Achtung! Datei wird eingelesen
renamingFile=Datei wird umbenannt
```

리소스 번들 연결
: 로거를 생성할 때 사용할 리소스 번들의 이름을 지정할 수 있다.
```java
// 로거 이름과 리소스 번들 이름을 함께 전달
Logger logger = Logger.getLogger("com.mycompany.myapp", "com.mycompany.logmessages");
```
: 이후에는 로그를 남길 때 메시지 대신 키를 사용한다
```java
// 실제로는 "Achtung! Datei wird eingelesen" 등으로 변환되어 기록됨
logger.info("readingFile");
```

Placeholders
: 로그 메시지 안에 파일 이름이나 숫자 같은 동적인 값을 넣어야 할 때가 많다. 이때는 placeholder 를 사용한다.
```java
readingFile=Reading file {0}.
renamingFile=Renaming file {0} to {1}.
```
: 값을 전달하기 위해 `log` 메서드를 사용한다.
```java
// 인자가 1개일 때
logger.log(Level.INFO, "readingFile", fileName);

// 인자가 여러 개일 때 (배열 사용)
logger.log(Level.INFO, "renamingFile", new Object[] { oldName, newName });
```

`logrb`
: Java 9 부터는 리소스 번들의 이름이 아니라 리소스 번들 객체 자체를 직접 전달할 수 있는 `logrb` 메서드가 강화되었다.  이 메서드는 가변인자를 지원하여 배열을 만들지 않고도 편리하게 값을 넘길수 있다.
```java
// 배열 생성 없이 값을 나열 가능
logger.logrb(Level.INFO, bundle, "renamingFile", oldName, newName);
```



## Handlers

핸들러
: Logger는 로그를 생성하는 역할을 할 뿐, 실제로 화면에 출력하거나 파일에 쓰는 역할은 Handler가 담당한다
- 로거는 로그 레코드를 자신의 핸들러뿐만 아니라 부모의 핸들러에게도 보낸다
- 루트 로거는 (`""`) 기본적으로 `ConsoleHandler`를 가지고 있어서 `System.err` 스트림으로 로그를 출력한다.
: 레벨 검사
- 로그가 최종적으로 기록되려면 로거의 레벨과 핸들러의 레벨을 모두 통과해야 한다.
- ex) 로거는 `FINE` 인데 콘솔 핸들러가 `INFO` 라면, `FINE` 로그는 콘솔에 출력되지 않는다.

핸들러 설정 및 중복 출력 방지
: 특정 로거에만 별도의 핸들러를 붙여서 세밀하게 제어할 수 있다. 이때 주의할 점은 Double Logging 문제이다.
```java
Logger logger = Logger.getLogger("com.mycompany.myapp");
logger.setLevel(Level.FINE); // 로거 레벨 설정

// 부모 핸들러(루트의 ConsoleHandler) 사용 중지 -> 중복 출력 방지
logger.setUseParentHandlers(false);

// 새로운 콘솔 핸들러 생성 및 부착
var handler = new ConsoleHandler();
handler.setLevel(Level.FINE); // 핸들러 레벨 설정
logger.addHandler(handler);
```
- `setUseParentHandlers(false)` : 이를 설정하지 않으면, 새로 만든 핸들러에서 한 번 출력되고, 부모 핸들러(루트)에서 또 한번 출력되어 같은 로그가 두 번 나온다

FileHandler
: 로그를 파일로 저장하며, 파일이 너무 커지지 않도록 관리하는 기능도 제공한다.
- `new FileHandler()` -> 사용자 홈 디렉토리의 `javaN.log`(N은 중복 방지 번호)에 저장, XML 형식으로 저장된다.
: 파일 이름을 지정할 때 특수 패턴 변수를 사용할 수 있다.

| **변수** | **설명**                                        |
| ------ | --------------------------------------------- |
| `%h`   | 사용자 홈 디렉토리 (User's home directory)            |
| `%u`   | 중복 파일 해결을 위한 고유 번호 (Unique number)            |
| `%g`   | 로그 회전(Rotation)을 위한 생성 번호 (Generation number) |
| `%%`   | 퍼센트(`%`) 문자 자체                                |
- ex) `%h/myapp.log` -> 홈 디렉토리의 `myapp.log` 파일에 저장
: Rotation, append
- `append`: 여러 어플리케이션이 하나의 로그 파일을 공유하거나, 껐다 켜도 로그를 유지하려면 `append` 플래그를 켜야 한다.
- `rotation` : 파일 크기가 일정 수준을 넘으면 새 파일을 만든다.
	- `myapp.log.0`, `myapp.log.1`, `myapp.log.2` (최신 -> 과거) 순으로 관리된다. 제한을 넘어가면 가장 오래된 파일이 삭제된다.


Custom Handlers
: 기본 제공 핸들러(`ConsoleHandler`, `FileHandler`, `SocketHandler`) 외에 핸들러를 따로 만들 수 있다. 교재에서는 로그를 GUI 창에 보여주는 `WindowHandler` 를 예시로 든다.
-  만들 때 `StreamHandler`, `Handler` 클래스를 상속받아 구현한다.
```java
class WindowHandler extends StreamHandler {
    public WindowHandler() {
        // 출력 스트림을 JTextArea로 연결하는 로직 
    }

    // 중요: publish 오버라이딩
    public void publish(LogRecord record) {
        super.publish(record);
        flush(); // 버퍼를 즉시 비워야 함
    }
}
```
- 주의사항: `StreamHandler` 는 기본적으로 효율성을 위해 로그를 Buffering 한다. 버퍼가 꽉 찰때까지 출력을 안하기 때문에, 프로그램이 비정상 종료되면 마지막 로그들을 잃어버릴 수 있다. 따라서 `publish` 메서드를 오버라이딩 하여 매번 `flush()`를 호출하는 것이 좋다.

## Filters

로거와 핸들러는 로그 레벨을 기준으로 1차적인 필터링을 수행한다.
`Filter` 인터페이스를 사용하여 레벨 이외의 조건(ex: 특정 단어 포함, 특정 시가대, 특정 사용자 등)으로 로그를 선별할 수 있다.
- 이는 Logger, Handler 모두에 적용될 수 있다.

구현 방법
: `java.util.logging.Filter` 인터페이스를 구현하고, `isLoggable`을 정의하면 된다.
```java
public interface Filter{
	boolean isLoggable(LogRecord record);
}
```
- 입력: `LogRecord` 객체(로그의 모든 정보) -> 출력: `true`/`false` = 기록/무시

ex) 메서드 추적용 로그 (`ENTRY`, `RETURN`으로 시작하는 메시지)만 남기고 나머지는 무시하는 필터
```java
class TraceFilter implements Filter {
    @Override
    public boolean isLoggable(LogRecord record) {
        String message = record.getMessage();
        // 메시지가 없으면 false
        if (message == null) return false;
        
        // ENTRY 또는 RETURN으로 시작하는 경우만 true
        return message.startsWith("ENTRY") || message.startsWith("RETURN");
    }
}
```

필터 장착
: `setFilter` 메서드를 사용하여 로거 또는 핸들러에 필터를 장착한다.
```java
Logger logger = Logger.getLogger("com.mycompany.myapp");
logger.setFilter(new TraceFilter());
```
- 주의사항: 하나의 로거나 핸들러에는 동시에 최대 하나의 필터만 장착할 수 있다. 만약 여러 조건을 적용하고 싶다면, 하나의 필터 클래스 안에서 모든 조건을 and 연산 등으로 묶어서 처리해야 한다.



## Formatters

포맷터
: 핸들러가 로그를 어디에 쓸지 결정한다면, Formatter는 로그를 어떤 모양으로 쓸지를 결정한다
- 기본 포맷터:
	- `ConsoleHandler` : `SimpleFormatter`를 사용하여 일반 텍스트로 출력한다
	- `FileHandler` : `XMLFormatter`를 사용하여 XML 구조로 출력한다.
- 다른 형식이 필요하다면 `Formatter` 클래스를 상속받아 만들 수 있다.
	- 다음은 사용자 정의 포맷터를 만들 때 오버라이딩해야 할 핵심 메서드들이다.
	- `format(LogRecord record)` (필수)
		- 로그 레코드를 받아서 최종 문자열을 반환한다. 날짜, 레벨, 클래스 이름 등은 `record` 객체에서 꺼내온다
		-  메시지 부분은 직접 파싱하지 말고 `formatMessage(record)`를 호출하면, localization(언어변환) 과 매개변수 치환을 알아서 해준다.
	- `getHead(Handler h)`, `getTail(Handler h)` (선택)
		- 로그 파일의 시작 부분과 끝 부분을 정의한다. XML 이나 HTML 처럼 여닫는 태그가 필요한 형식에서 유용하다.

ex) HTML 포맷터 : HTML 테이블 형태로 로그를 남기는 포맷터 예시
```java
class MyHtmlFormatter extends Formatter {
    // 1. 파일의 시작 (테이블 열기)
    @Override
    public String getHead(Handler h) {
        return "<HTML><HEAD><TITLE>My Log</TITLE></HEAD><BODY><TABLE border='1'>\n" +
               "<TR><TH>Level</TH><TH>Time</TH><TH>Message</TH></TR>\n";
    }

    // 2. 각 로그 레코드 포맷팅 (테이블 행 추가)
    @Override
    public String format(LogRecord record) {
        return "<TR>" +
               "<TD>" + record.getLevel() + "</TD>" +
               "<TD>" + new Date(record.getMillis()) + "</TD>" +
               "<TD>" + formatMessage(record) + "</TD>" + // formatMessage 사용!
               "</TR>\n";
    }

    // 3. 파일의 끝 (테이블 닫기)
    @Override
    public String getTail(Handler h) {
        return "</TABLE></BODY></HTML>\n";
    }
}
```

포맷터 장착
: 만들어진 포맷터를 핸들러에 장착하려면 `setFormatter` 메서드를 사용한다.
```java
var handler = new FileHandler("%h/myapp.html");
handler.setFormatter(new MyHtmlFormatter()); // 포맷터 교체
logger.addHandler(handler);
```




## A Logging Recipe

로거 이름
- 메인 애플리케이션 패키지 이름을 로거 이름으로 사용하셈...
- 로그를 많이 남기는 클래스에는 `static` 필드로 선언해 두는 것이 좋다
```java
private static final Logger logger = Logger.getLogger("com.mycompany.myprog");
```

기본 설정 덮어쓰기
- 기본 설정(ConsoleHandler, INFO 레벨)은 부족할 수 있다.
- 사용자가 별도의 설정 파일을 지정하지 않았다면, 프로그램 시작 시점(`main`)에서 FileHandler 를 직접 등록하라

마음껏 로그 남기기
- INFO, WARNING, SEVERE : 사용자에게 의미있는 정보만 남긴다. 
- FINE : 개발자를 위한 정보.









































