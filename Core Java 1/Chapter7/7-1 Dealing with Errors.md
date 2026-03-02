
오류 발생 시 프로그램이 수행해야 할 두가지 핵심 목표:
- Return to a safe state: 사용자가 다른 명령을 계속 실행할 수 있도록
- Graceful termination: 모든 작업을 저장한 후 안전하게 프로그램을 종료
하지만, 오류를 감지하는 코드와 그 데이터를 복구하거나 저장하는 코드는 서로 멀리 떨어져 있는 경우가 많아 이를 구현하기 쉽지 않다. 
예외 처리의 주된 목적은 오류가 발생한 시점에서 Error Handler로 제어권을 넘기는 것이다.

프로그래머는 예외적인 상황을 처리하기 위해 다음과 같은 네 가지 유형의 문제를 고려해야 한다.
Types of Problems:
- User input error
	- 사용자는 단순한 오타뿐만 아니라, 지시 사항을 따르지 않고 독자적인 행동을 하기도 한다.
	- ex) 사용자가 문법적으로 잘못된 URL을 입력하여 연결을 시도. 이 경우 코드가 문법검사를 하지 않으면 네트워크 계층에서 불만을 제기하게 된다.
- Device errors
	- 하드웨어는 항상 의도한 대로 작동하지 않으며, 작업 도중에 실패할 수 있다.
	- ex) 프린터가 꺼져 있거나, 인쇄 도중 종이가 떨어지는 경우 등
- Physical Limitations
	- 시스템 자원의 한계
	- ex) 디스크 용량이 가득 차거나, 사용 가능한 메모리가 부족한 경우
- Code errors
	- 메서드가 잘못된 값을 내놓거나 다른 메서드를 잘못 사용하는 등의 버그
	- ex) 유효하지 않은 배열 인덱스 계산, 해시 테이블에 없는 항목 검색 시도 등

전통적으로는 메서드에서 오류가 발생하면 특수 오류코드를 반환하여 호출한 메서드가 이를 분석했다.
- 파일 읽기에서 `-1`을 반환하여 EOF를 알리거나, 참조 반환에서 오류 발생하면 `null` 반환
- 하지만, 항상 오류 코드를 반환할 수 있는 것은 아니다. 
	- ex) 정수를 반환하는 메서드의 경우, 오류를 나타내기 위해 -1을 반환하려 해도, -1 자체가 계산된 유효한 결과값일 수 있다. 이 경우 정상적인 값과 오류 코드를 구별할 수 없다.

Java's Solution
: 자바는 메서드가 정상적으로 작업을 완료할 수 없을 때 사용할 수 있는 Alternative exit path(대체 종료 경로)를 제공한다.
- 1. 메서드는 값을 반환하지 않고, 오류 정보를 캡슐화한 객체를 throw
- 2. 메서드는 즉시 종료되며, 값을 반환하지 않는다.
- 3. 메서드를 호출한 코드의 다음 줄에서 실행이 재개되는 것이 아니라, 예외 처리 메커니즘이 해당 오류를 처리할 수 Exception handler를 찾는다.



## The Classification of Exceptions


Exception Hierarchy
: Java에서 모든 예외 객체는 `Throwable` 클래스에서 파생된 인스턴스이다. Java의 예외 계층 구조는 `Throwable`을 루트로 하여 크게 두가지로 나뉜다.
- 1. Error
	- Java 런타임 시스템 내부의 오류나 자원 고갈 상황
	- 프로그래머가 이 유형의 객체를 throw 하거나 처리하려고 해서는 안된다.
	- 내부 오류가 발생하면 사용자에게 알리고 프로그램을 정상적으로 종료(Graceful termination)하도록 노력하는 것 외에는 할 수 있는 일이 거의 없다.
- 2. Exception
	- `Exception`은 두가지로 나뉜다.
		- 1. `RuntimeException`에서 파생된 예외
		- 2. 그렇지 않은 예외


RuntimeExcetpion과 기타 예외
: 두가지를 구분하는 일반적인 규칙은 누구의 탓인가에 달려있다.
- 1. RuntimeException 
	- 코드를 올바르게 작성했다면 발생하지 않았을 문제들, 사전에 검사를 통해 방지할 수 있다.
	- 형변환이 잘못되거나, 배열 범위를 벗어나거나, null 포인터 접근 등
- 2. Non-RuntimeException
	- 프로그램 로직 자체는 문제가 없으나, I/O 오류 등 외부 상황에 의해서 발생하는 문제
	- 파일의 끝을 지나쳐서 읽으려 하거나, 존재하지 않는 파일을 열려고 하거나, 존재하지 않는 클래스를 문자열로 찾으려 할 때


Checked, Unchecked
: 예외를 처리 여부에 따라 Checked/Unchecked 예외로 구분한다.

| 분류                  | 포함되는 예외 클래스                  | 컴파일러의 동작                                 |
| ------------------- | ---------------------------- | ---------------------------------------- |
| Unchecked Exception | `Error` + `RuntimeException` | 예외 처리기(Handler) 작성을 강제하지 않음.             |
| Checked Exception   | 그 외 모든 `Exception`           | **반드시** 예외 처리기를 제공해야 함. (안 하면 컴파일 에러 발생) |


## Declaring Checked Exception

메서드는 이런 상황에서 실패할 수도 있음을 알려야 한다. 이를 통해 메서드를 사용하는 쪽에서 잠재적인 위험에 대비할 수 있다.

ex) FileInputStream 생성자
표준 라이브러리의 `FileInputStream` 클래스 생성자는 다음과 같다.
```java
public FileInputStream(String name) throws FileNotFoundException
```
- 이 생성자는 `String`을 받아 `FileInputStream` 객체를 생성하려고 시도하지만, 만약 파일이 없다면 객체를 초기화하지 않고 `FileNotFoundException` 객체를 던질 수 있다.
- 예외가 발생하면 즉시 실행을 멈추고 Exception Handler를 찾는다.

예외가 발생하는 4가지 상황
- 1. 위험한 메서드 호출 : Checked Exception 을 던지는 다른 메서드를 호출
- 2. 직접 투척: 오류를 감지하여 `throw`문으로 직접 Checked Exception을 던질 때
- 3. 프로그래밍 오류: 배열 인덱스 오류등으로 인해 Unchecked Exception이 발생할 때
- 4. 내부오류: jvm 이나 런타임 라이브러리에서 내부적인 `Error`가 발생할 때
1,2번의 경우 반드시 메서드 헤더에 `throws`를 사용하여 예외를 선언해야한다.


Checked Exception 은 나열해야한다.
ex)
```java
public Image loadImage(String s) throws FileNotFoundException, EOFException {...}
```

Unchecked Exception은 선언하지 말아야한다.
- ex) `OutofMemoryError, ArrayIndexOutOfBoundsException, NullPointerException` 등등

상속의 경우
: 메서드를 Overriding 할 때, 예외 선언에는 중요한 제약이 있다.
- 자식의 메서드는 부모의 메서드보다 더 일반적인 예외를 던질 수 없다.
	- 부모가 던지는 예외의 하위 클래스를 던지거나, 아예 예외를 던지지 않는 것은 괜찮다. 하지만, 부모가 던지지 않는 Checked Exception을  자식이 새로 추가해서 던질 수 없다.


예외의 다형성 (Polymorphism)
: 특정 예외 클래스를 던진다고 선언하면, 그 클래스의 모든 하위 클래스도 던져질 수 있음을 의미
- 예: `throws IOException`이라고 선언하면, 실제로는 `FileNotFoundException`이나 `EOFException` 같은 하위 객체가 던져질 수 있다.



## How to Throw an Exception

데이터를 읽어들이는 `readData` 라는 메서드가 있다고 가정하자
- 파일 헤더에 `Content-length: 1024` 라고 명시되어 있어, 1024 바이트를 읽기를 기대한다.
- 하지만 733바이트만 읽었는데 EOF에 도달 --> 처리를 중단하고 예외를 던지기로 함

3 Steps
- 1. 적절한 예외 클래스 찾기: 
	- `IOException` 계열이 적절해 보임. 실제로도 Java API 문서에서 이는 입력 도중 예상치 못하게 EOF에 도달했음을 알리는 예외이다.
- 2. 그 클래스의 객체 생성하기:
	- `new EOFException()`
- 3. Throw

구문
- `throw new EOFException();` 혹은
- `var e = new EOFException(); throws e;`

ex)
```java
String readData(Scanner in) throws EOFException // 1. 예외 선언
{
    . . .
    while (. . .)
    {
        if (!in.hasNext()) // 예상보다 일찍 파일 끝에 도달함
        {
            if (n < len) {
                // 2. 비정상 상황 감지 시 예외 투척
                throw new EOFException();
            }
        }
        . . .
    }
    return s;
}
```


Describing the Condition
: `EOFException` 을 포함한 대부분의 예외 클래스는 String을 인자로 받는 생성자를 하나 더 가지고 있다. 이를 이용하여 오류 상황을 더 구체적으로 설명할 수 있다.
```java
String gripe = "Content-length: " + len + ", Received: " + n;
throw new EOFException(gripe);
```
--> 오류로그를 볼때 뭐가 문제였는지 정확히 알 수 있다.



## Creating Exception Classes

사용자 정의 예외를 만드는 경우
: 수만은 표준 예외가 있지만, 모든 상황을 커버할 수는 없다.

작성 방법
- 1. 상속
	- `Exception` 클래스를 상속받거나, 성격에 맞는 `Exception`의 자식 클래스(ex: `IOException`)를 상속받는다.
- 2. 생성자
	- 관례적으로 두가지 생성자를 정의한다.
	- 1. 기본 생성자, 2. 메시지 생성자.(상세 오류 메시지를 받아 상위 클래스로 전달)

ex) `FileFormatException`
```java
class FileFormatException extends IOException
{
   // 1. 기본 생성자
   public FileFormatException() {}

   // 2. 상세 메시지를 받는 생성자
   public FileFormatException(String gripe)
   {
      super(gripe); // 상위 클래스(Throwable)의 생성자를 호출하여 메시지 저장
   }
}
...
...
...
// 메서드 헤더에 사용자 정의 예외를 선언
String readData(Scanner in) throws FileFormatException
{
   . . .
   while (. . .)
   {
      if (ch == -1) // 예상치 못한 파일 끝(EOF) 도달
      {
         if (n < len)
            // 사용자 정의 예외 객체 생성 및 던지기
            throw new FileFormatException();
      }
      . . .
   }
   return s;
}

```

 
`java.lang.Throwable`
: Throwable 클래스의 주요 메서드
- `Throwable()` : 상세 메시지 없이 새로운 Throwable 객체를 생성
- `Throwable(String message)` : 지정된 상세 메시지를 담은 Throwable 객체를 생성
- `String getMessage()` : Throwable 객체에 저장된 상세 메시지를 가져옴




 

















