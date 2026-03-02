
## Overloading

- 이름은 똑같지만 파라미터의 개수나 타입이 다른 메서드를 여러 개 만드는 것.
    - 생성자(`Constructor`)뿐만 아니라 일반 메서드도 모두 오버로딩 가능함.
        
- Overloading Resolution:
    - 메서드를 호출할 때 넘겨준 값들의 타입을 보고, 컴파일러가 알아서 가장 잘 맞는 메서드를 골라줌.
    - 만약 딱 맞는 게 없거나, 애매하면(Ambiguous) 컴파일 에러 남.
    
- Method Signature
    - 메서드를 구분하는 기준은 메서드 이름 + 파라미터 타입 목록 = 시그니처
    - 예: `indexOf(int)`와 `indexOf(String)`은 시그니처가 달라서 다른 메서드 취급함.
        
- 주의
    - .return type은 시그니처에 포함 안 됨.
    - 즉, 이름과 파라미터는 똑같은데 리턴 타입만 다르게 만드는 건 불가능함. '


## Default Field Initialization

자동 초기화 
생성자에서 필드 값을 따로 설정 안 하면 기본값으로 채워짐
- **숫자:** `0`
- **boolean:** `false`
- **객체 참조:** `null`

지역 변수와 차이점 
- **지역 변수 (메서드 내부):** 절대 자동 초기화 안 됨. 개발자가 **반드시** 값을 넣어줘야 함. 안 그러면 에러
- **필드 (클래스 멤버):** 값 안 넣어도 자동으로 위의 기본값(`0`, `null` 등)이 들어감.

웬만하면 명시적으로 초기화하는 게 좋음.


## Constructor with no Arguments

파라미터 없이 `new Employee()` 처럼 호출하는 생성자
보통 객체를 적절한 '기본값' 상태(예: 빈 문자열, 현재 날짜 등)로 초기화할 때 사용

자동 생성 규칙 
- **생성자가 아예 없으면:** 컴파일러가 파라미터 없는 기본 생성자 만들어줌. (이때 필드는 0, false, null로 초기화됨).
    
- **생성자가 하나라도 있으면:** 파라미터 있는 생성자를 하나라도 작성하는 순간, 기본 생성자는 제공되지 않음.

파라미터 있는 생성자도 쓰고 싶고, 기본 생성자도 쓰고 싶으면, 무인자 생성자를 반드시 직접 코드로 작성해줘야 함.


## Explicit Field Initialization

클래스 안에서 필드를 선언함과 동시에 값을 대입하는 것.

`private String name = "";` 처럼 변수 선언부에 바로 씀.
- 생성자가 실행되기 전에 이 초기화 코드가 먼저 실행됨

단순한 상수 값뿐만 아니라, method 호출 결과를 바로 초기값으로 넣을 수도 있음
- `private int id = advanceId();`



## Calling Another Constructor


- **`this`의 두 번째 용도:** 원래는 현재 객체를 가리키지만, 생성자 내부에서 `this(...)` 형태로 쓰면 **같은 클래스 내의 다른 생성자**를 호출한다는 뜻
    
- 반드시 생성자의 가장 첫 번째 문장(First statement)으로 와야 함.
    
- 장점: 초기화 로직이 여러 생성자에 분산되지 않게 하고, 공통된 초기화 코드를 한 곳에 작성해서 재사용할 수 있음 (코드 중복 제거).

``` java
public Employee(double s) {
    //다른 생성자인 Employee(String, double)을 호출함
    // 이 줄이 반드시 가장 먼저 와야 함
    this("Employee #" + nextId, s); 
    
    nextId++; // 그 다음 로직 수행
}
```
`new Employee(60000)`을 호출하면 -> `this(...)`를 통해 이름과 급여를 받는 다른 생성자가 먼저 실행되고 -> 다시 돌아와서 `nextId++`가 실행됨.



## Initialization Blocks

- Object Initialization Block)
    - `{ ... }` 
    - 객체가 생성될 때마다 생성자 본문보다 먼저 실행됨.
    - 용도: 모든 생성자에서 공통적으로 수행해야 하는 초기화 로직이 있을 때 씀 (하지만 보통은 그냥 생성자 안에 넣는 게 더 직관적이라 잘 안 씀).

- 정적 초기화 블록 (Static Initialization Block):
    - `static { ... }`
    - 클래스가 메모리에 처음 로딩될 때 딱 한 번 실행됨.
    - 용도: `static` 필드를 복잡한 로직(예: 난수 생성, 파일 로딩 등)으로 초기화해야 할 때 필수적임.

```java
class Employee
{
	private static int nextId;
	private int id;
	private String name;
	private double salary;
	
	{
		id=nextId;
		nextId++;
	}
	
	static {  
	    Random generator = new Random();  
	    nextId = generator.nextInt(1000); // 0 ~ 999 사이 난수  
	}
}
```



## Object Destruction and the finalize Method

C++, have explicit destructor methods for any cleanup code that may be needed when an object is no longer used.

**Java does automatic garbage collection, manual memory reclamation is not needed, so Java does not support destructors.**

**메모리 외 자원 관리:** 파일, 소켓, DB 연결 같은 '시스템 자원'은 GC가 알아서 닫아주지 않음. 따라서 이건 개발자가 직접 반납해줘야 함.

자원 정리 방법
- **`close()`**
    - 객체 사용이 끝나면 즉시 `close()` 같은 메서드를 호출해서 자원을 반납하도록 구현해야 함. (보통 Chapter 7에서 배울 `try-with-resources` 문과 함께 사용됨).
        
- **`Runtime.addShutdownHook`:**
    - 프로그램이 종료(VM 종료)될 때 실행하고 싶은 작업이 있으면 여기에 등록함.
        
- **`Cleaner` 클래스 (Java 9 이상):**
    - 객체가 GC 대상이 될 때 수행할 동작을 등록할 수 있음. 하지만 실행 시점이 정확하지 않으므로 꼭 필요한 특수 상황 아니면 잘 안 씀.


`finalize()` 메서드는 절대 쓰지 말 것.
- 과거에는 객체가 소멸되기 직전에 호출된다고 했지만, **언제 실행될지 보장할 수 없음** (프로그램 끝날 때까지 안 불릴 수도 있음).
    
- 성능 문제도 심각함.
    
- 현재 Java에서 공식적으로 **Deprecated(사용 중단 권고)** 상태임.



















