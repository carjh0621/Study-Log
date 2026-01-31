
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











