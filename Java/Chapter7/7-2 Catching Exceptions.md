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
- Catching : 오류가 발생하면 그 자리에서 로그를 남기거나 대응
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
























