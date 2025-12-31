
## structure & rules

- **인스턴스 필드 (Instance Fields):** 객체의 데이터 상태 (예: 이름, 월급). 보통 `private`으로 숨긴다. (C `struct`의 멤버 변수)
    
- **생성자 (Constructor):** 객체가 `new`로 힙에 올라갈 때 딱 한 번 실행되는 초기화 코드.
    
- **메서드 (Methods):** 데이터를 조작하거나 조회하는 함수들.
```java
class ClassName {    
	field1    
	field2    
	. . .    
	constructor1    
	constructor2    
	. . .    
	method1    
	method2    
	. . . }
```

```java
예시:
class Employee {    
	// instance fields    
	private String name;    
	private double salary;    
	private LocalDate hireDay;    
	// constructor    
	public Employee(String n, double s, int year, int month, int day)    
	{       
		name = n;       
		salary = s;       
		hireDay = LocalDate.of(year, month, day);    
	}    
	// a method    
	public String getName()    
	{       
		return name;    
	}    
	// more methods    . . . 
}
```

파일과 클래스 규칙 

- **1파일 1 public 클래스:** 소스 파일 하나(`FileName.java`)에는 `public` 클래스가 단 하나만 있어야 하고, 파일명은 그 클래스 이름과 같아야 함.
    
- **비공개 클래스:** `public`이 아닌 클래스(`class Employee`)는 같은 파일 안에 여러 개 만들어도 됨.
    
- **컴파일 결과:** `javac EmployeeTest.java`를 실행하면, 파일은 하나였어도 바이트코드 파일은 클래스별로 각각 생성됨. (`EmployeeTest.class`, `Employee.class` 두 개가 생김)


## using multiple files

Many programmers prefer to put each class into its own source file. 
For example, you can place the `Employee` class into a file `Employee.java` and the `EmployeeTest` class into `EmployeeTest.java`.

You can invoke the Java compiler with a wildcard: 
- `javac Employee*.java` 
- Then, all source files matching the wildcard will be compiled into class files. 

Or, you can simply type 
- `javac EmployeeTest.java`

Java compiler sees the Employee class being used inside EmployeeTest.java, it will look for a file named Employee.class. 
If it does not find that file, it automatically searches for Employee.java and compiles it. 
Moreover, if the timestamp of the version of Employee.java that it finds is newer than that of the existing Employee.class file, the Java compiler will automatically recompile the file.

`javac`의 의존성 추적 (Make-like Behavior)

`javac`는 단순한 번역기가 아니라, **빌드 도구의 기능**을 일부 내장하고 있음. `EmployeeTest.java`를 컴파일할 때 다음과 같은 로직이 수행됨.
1. **심볼 탐색:** 코드 안에서 `Employee` 클래스를 발견.
2. **클래스 파일 확인:** `Employee.class` 파일이 이미 있는지 확인.
3. **소스 파일 탐색:** 없다면 `Employee.java`를 찾아 자동으로 컴파일.
4. **타임스탬프 대조 (Dependency Check):**
    - `Employee.class`가 있더라도, 소스 파일(`Employee.java`)의 **수정 시간(Timestamp)**이 더 최신이라면?
    - 변경된 내용이 반영되지 않았다고 판단하고 **자동으로 재컴파일**



## Constructor

A constructor can only be called in conjunction with the new operator. 
You can’t apply a constructor to an existing object to reset the instance fields. For example,
- `james.Employee("James Bond", 250000, 1950, 1, 1) // ERROR`
is a compile-time error.

- A constructor has the same name as the class. 
- A class can have more than one constructor. 
- A constructor can take zero, one, or more parameters. 
- A constructor has no return value. 
- A constructor is always called with the new operator.

caution (Shadowing Bug)
```java
public Employee(String n, double s, . . .) 
{ 
	String name = n; // ERROR 
	double salary = s; // ERROR   
	. . . 
}
```
The constructor declares local variables name and salary. 
These variables are only accessible inside the constructor. 
They shadow the instance fields with the same name.
=> 필드가 초기화가 안됨

## Declaring with var

`Employee harry = new Employee("Harry Hacker", 50000, 1989, 10, 1);` 
you simply write 
- `var harry = new Employee("Harry Hacker", 50000, 1989, 10, 1);`

**제약 사항:**
- **지역 변수(Local Variable)에만 사용 가능.** (메서드 내부 변수)
- 클래스 필드(Member Variable)나 메서드 파라미터에는 절대 못 씀.
- 초기값이 반드시 있어야 함. (컴파일러가 추론할 단서가 필요함)
	- var i = 10; // int로 추론됨 
	- var d = 10.0; // double로 추론됨
	- var harry = new Employee("Harry", 50000); 
		- //오른쪽이 Employee니까 harry는 Employee 타입 추론


## null References

If you apply a method to a null value, a `NullPointerException` occurs.
```java
LocalDate rightNow = null; 
String s = rightNow.toString(); // NullPointerException
```

When you define a class, it is a good idea to be clear about which fields can be null. 
In our example, we don’t want the name or hireDay field to be null.

There are two solutions. The “permissive” approach is to turn a null argument into an appropriate non-null value:
`if (n == null) name = "unknown"; else name = n; `

The Objects class has a convenience method for this purpose:
```java
public Employee(String n, double s, int year, int month, int day) {    
	name = Objects.requireNonNullElse(n, "unknown");    
	. . . 
}
```

The “tough love” approach is to reject a null argument:
``` java
public Employee(String n, double s, int year, int month, int day) {    
	name = Objects.requireNonNull(n, "The name cannot be null");    
	. . . 
}
```
there are two advantages: 
1. The exception report has a description of the problem. 
2. The exception report pinpoints the location of the problem. Otherwise, a NullPointerException would have occurred elsewhere, with no easy way of tracing it back to the faulty constructor argument.


## Implicit and Explicit Parameters

``` java
public void raiseSalary(double byPercent) {    
	double raise = salary * byPercent / 100;    
	salary += raise; 
}
```
Consider the call
- `number007.raiseSalary(5);`
More specifically, the call executes the following instructions:
```java
double raise = number007.salary * 5 / 100; 
number007.salary += raise;
```

- **명시적 매개변수 (Explicit):** 괄호 안에 있는 숫자 `5`
- **암시적 매개변수 (Implicit):** 메서드 이름 앞에 있는 객체 `number007`
    - 메서드 내부에서는 이 암시적 파라미터를 **`this`**라는 키워드로 참조

``` java
public void raiseSalary(double byPercent) {    
	double raise = this.salary * byPercent / 100;    
	this.salary += raise; 
}
```
it clearly distinguishes between instance fields and local variables.


## Encapsulation 

caution
```java
class Employee {    
	private Date hireDay;    
	. . .    
	public Date getHireDay() 
	{       
		return hireDay; // BAD    
	}    
	. . . 
}
```
`Employee` 클래스 안에 `private Date hireDay;`가 있음.
`getHireDay()`가 `hireDay` 필드의 **참조값(Reference/Pointer)**을 그대로 리턴함.

**결과:** 외부에서 받은 참조값을 통해 `Date` 객체의 `setTime()`을 호출하면, **`Employee` 내부의 `private` 데이터가 바뀜.** 
- ex)
	- harry -> employee -> date
	- `Date d = harry.getHireDay();`
		- d->date(harry's)

use immutable or return copy
```java
class Employee {    
	private Date hireDay;    
	. . .    
	public Date getHireDay() 
	{       
		return (Date) hireDay.clone; // OK    
	}    
	. . . 
}
```



## access privileges

What people often find surprising is that a method can access the private data of all objects of its class.
For example, consider a method `equals` that compares two employees.
```java
class Employee {   
	 . . .    
	public boolean equals(Employee other)    
	{       
		return name.equals(other.name);    
	} 
}
```

`if (harry.equals(boss)) . . .`

This method accesses the private fields of harry, which is not surprising. It also accesses the private fields of boss.
This is legal because boss is an object of type Employee, and a method of the Employee class is permitted to access the private fields of any object of type Employee.


## Private method

오직 클래스 내부의 다른 메서드들만 이 `private` 메서드를 호출해서 사용함.
- **Public 메서드:** 한번 공개하면 "약속(Contract)"이 돼버려서, 함부로 지우거나 이름을 바꾸면 내 라이브러리를 쓰는 다른 사람 코드가 다 터짐. 
    
- **Private 메서드:** 외부에서 어차피 못 봄. 내가 나중에 내부 구현을 뜯어고치면서 **이 메서드가 필요 없어지면 그냥 지워버려도 아무 문제가 없음.** 유지보수가 훨씬 자유로움.



## Final Instance

final field
```java
class Employee {    
	private final String name;    
	. . . 
}
```
You can define an instance field as final. Such a field must be initialized when the object is constructed.
Afterwards, the field may not be modified again.

final reference
- `final String name = "Bond";`
    - `String`은 불변(Immutable) 객체라 내용도 못 바꾸고, 참조도 못 바꿈. 완벽한 상수.
        
- `final StringBuilder evaulations = new StringBuilder();`
    - 여기서 `final`은 evaulations라는 변수가 가리키는 주소(포인터)를 잠그는 것, 힙 메모리에 있는 StringBuilder 객체의 내용을 잠그는 게 아님.
    - 즉, `evaluations = new ...` (재할당)은 불가능
    - 하지만 `evaluations.append(...)` (내용 수정)은 .가능.