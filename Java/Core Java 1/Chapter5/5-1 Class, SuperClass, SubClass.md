
## Subclasses

Java에서는 `extends` 키워드를 사용해 상속을 정의함.

Java의 모든 상속은 `public` 상속임. (private, protected 상속 x)

**기존 클래스** : superclass, base class, parent class
**새로운 클래스** : subclass, derived class, child class

역설적이게도 기능 면에서는 **서브클래스가 슈퍼클래스보다 더 많은 기능과 데이터**를 가짐.

- 서브클래스는 슈퍼클래스의 메서드(`getName` 등)를 그대로 물려받아 사용 가능.
- 서브클래스에만 필요한 새 필드(`bonus`)와 메서드(`setBonus`)를 추가하여 확장함.

**Private 멤버의 상속 **
- 슈퍼클래스의 `private` 필드는 서브클래스에서 직접 접근 불가능.
- 하지만 서브클래스 객체 내부에 해당 데이터 공간은 존재함. (접근 권한만 없을 뿐 데이터는 가짐)

공통 기능은 슈퍼클래스에, 특수한 기능은 서브클래스에 배치함.

*`record` 타입은 상속을 하거나 받을 수 없음.

```java
public class Manager extends Employee
{
	...
} 
```



## Overriding Methods

SuperClass 의 method가 Subclass의 목적에 맞지 않을 때 재정의 하는 것. 

교과서의 예제 클래스인 `Employee` 의 서브클래스인 `Manager` 의 급여는 단순 기본급이 아니라, **기본급 + 보너스**  이다. 그러므로, `getSalary()` 메서드를 새로 작성해야함.

**주의**
- **private field 직접 접근 불가**
	- `return salary + bonus;` -> error 
	- `salary` 는 employee의 private field. manager에서 직접 건드리지 못한다.
- **무한 재귀 호출**
	- `return getSalary() + bonus;` -> Manager의 `getSalary()` 를 계속 호출하게 된다.

`super` 키워드
- 위의 문제점들을 해결하기 위해서는 `super.getSalary()` 라고 명시해야 한다.
- 이는 현재 클래스가 아닌 슈퍼 클래스(Employee)의 `getSalary()` 를 실행하라는 뜻. 
- `double baseSalary = super.getSalary(); return baseSalary + bonus;` 

`this` 와의 차이점
- `super` 는 `this` 처럼 객체의 주소를 담고 있는 참조가 아니다. 
- `super` 를 다른 변수에 대입할 수는 없다. 이는 컴파일러에게 부모 메서드를 호출하라고 명령하는 특수 명령어이다. 

상속을 통해 필드나 메서드를 추가하거나 오버라이딩할 수는 있음.
하지만 부모에게 물려받은 필드나 메서드를 삭제하는 건 불가능함.


## Subclass Constructors

**서브클래스 생성자(`super`)**
- 서브클래스 (`Manager`) 는 부모 (`Employee`) 의 `private` 필드를 직접 건드릴 수 없다.
- 그래서 부모 생성자를 호출해서 대신 초기화해야 한다.
- `super(name,salary,...);`
- **반드시 생성자의 첫 번째 문장에 와야한다.**
- `super` 를 명시적으로 쓰지 않으면, 컴파일러가 자동의 부모의 `no-arg constructor`를 호출한다. 만약 부모에 해당 생성자가 없으면 에러.

`this` vs `super`
- `this(...)` : 같은 클래스의 다른 생성자를 호출
- `super(...)` : 부모 클래스의 생성자 호출
- 둘 다 생성자의 첫 줄에만 올 수 있다. 그러므로 동시에 못 쓴다.

**다형성 (polymorphism)**
- 하나의 객체 변수가 여러 타입의 실제 객체를 참조할 수 있는 성질.
- ex) `Employee e = new Manager(...)` 가능

**동적 바인딩 (Dynamic Binding)**
- 컴파일 단계에서는 변수 타입(`Employee`) 만 보고 체크하지만, 실행(runtime)단계 에서는 실제 객체 (`Manager`) 가 뭔지 확인해서 그에 맞는 메서드를 찾아 실행하는것. 
- 교재의 예제를 보면, `staff` 배열에 `Employee`랑 `Manager`가 섞여 있어도, `e.getSalary()` 한 줄만 돌리면 알아서 각자 맞는 월급 계산법(보너스 포함 여부)이 적용됨.


## Inheritance Hierarchies

상속은 `Employee` -> `Manager`에서 끝나는 게 아님. `Manager` -> `Executive` 처럼 계속 이어질 수 있음. 이를 **상속 체인(Chain)** 이라고 함

`Employee` 아래에 `Manager` 말고 `Programmer`, `Secretary` 등 전혀 다른 형제 클래스들이 올 수 있음. 이걸 통틀어 **상속 계층(Hierarchy)** 이라고 부름

Java는 다중 상속 금지


## Polymorphism

**Is-a 관계** : Manager is and Employee 는 성립함. 반대는 안됨.

**Substitution Principle** : `Employee` 객체가 필요한 곳에는 언제든지 `Manager` 객체를쓸 수 있다.
- `Employee e = new Manager(...);` 가능
- `Manager m = new Employee(...);` 불가능

**주의점**
```java
Employee[] staff = new Employee[3];
staff[0] = new Manager(...) ; // 가능
...
staff[0].setBonus(5000); //에러발생
//setBonus는 Employee에 없고 Manager에만 존재
```
- 실제 객체는 `Manager`, 변수 `staff[0]` 의 타입이 `Employee`
- 컴파일러는 타입을 보고, `setBonus` 없다고 판단 --> 에러

`ArrayStoreException`
```java
Manager[] managers = new Manager[10];
Employee[] staff = managers; //가능 (배열도 객체라서)
...
staff[0] = new Employee(...); //컴파일은 통과, 하지만 실행 시 에러
```
- staff와 managers는 같은 힙 메모리를 가리킴
- 배열 자체는 매니저를 저장
- 이때 staff를 통해 Employee 넣으려고 함. 
- JVM이 실행 중에 감지하고, `ArrayStoreException`

---
```java
Employee[] staff = new Employee[10];
Manager[] bosses = staff; // compile error
```
이게 된다면, `bosses[0].setBonus(5000);` 같은 코드가 된다. 
하지만 실제 `bosses[0]`에 일반 Employee 가 있을 수도 있다. (bonus 없음)
컴파일 단계에서 차단

```java
Manager[] m = new Manager[10];
Employee[] e = m; // 가능 (참조만 복사)

e[0] = new Manager(); // 가능 (매니저 집에 매니저 넣음)
e[0] = new Employee(); // 런타임 에러! (매니저 집에 일반 직원 넣음 - 안됨)
```

```java
Employee[] e = new Employee[10]; // 힙에 직원용 아파트 생성

e[0] = new Manager(); // 가능 (매니저는 직원이니까)
e[0] = new Employee(); // 가능 (직원을 직원에 넣으니까)

// 덮어씌우기
e[0] = new Manager(); // 다시 매니저로 교체? 가능.
```

**부모 타입 변수 = 자식 객체 (가능)**
**자식 타입 변수 = 부모 객체 (불가능)**

---


## Understanding Method Calls

`x.f(args)`

1. 컴파일 타임
	1. 컴파일러는 변수 `x` 의 선언된 타입을 보고, 그 클래스와 부모 클래스에 있는 메서드 중 이름이 `f`인 것을 전부 모은다. (`private` 제외)
	2. Overloading Resolution
		- 모은 것들 중 `args` 타입과 제일 잘 맞는 메서드를 고른다.
		- 못 찾거나, 애매하면 컴파일 에러 발생.
		- 이 과정이 끝나면, 컴파일러는 호출한 메서드의 이름과 파라미터 타입(시그니처)을 확정한다.
2. 정적, 동적 바인딩
	1. 정적 바인딩:
		- `private, static, final` , 생성자
		- 오버라이딩 될 일이 없다. 바로 해당 메서드 실행하면 된다는 것을 앎
	2. 동적 바인딩:
		- 위를 제외한 나머지 모든 메서드
		- 실행 시점에 결정
3. Method Table, 성능 최적화
	- JVM은 메서드 테이블을 만들어서, 매번 메서드 호출시마다 상속 계층을 뒤지지 않도록 한다.
	- Employee 테이블 예시
	    - `getName()` -> `Employee.getName()`
	    - `getSalary()` -> `Employee.getSalary()`
        
	- Manager 테이블 예시    
	    - `getName()` -> `Employee.getName()` (상속받은 거 그대로 씀)
	    - `getSalary()` -> **`Manager.getSalary()`** (오버라이딩해서 바꿔치기됨)
	    - `setBonus()` -> `Manager.setBonus()` (새로 추가됨)

Method Signature:
- `이름` + `파라미터 타입 리스트`. (리턴 타입은 포함 안 됨)
- 오버라이딩 하려면 시그니처가 같아야 함.

Covariant Return Types:
- 오버라이딩 할 때 리턴 타입을 원래 리턴 타입의 자식 타입으로 바꾸는 건 허용됨.    
- 예: `Employee getBuddy()`를 `Manager getBuddy()`로 오버라이딩 가능. 

메서드는 부모가 `public`이면 자식도 무조건 `public`이어야 함.



## Final Class and Method

Final class : 더 이상 자식을 낳을 수 없음
- ex: `public final class Executive extends Manager { ... }`
- 클래스를 final로 선언하면, 그 안의 모든 메서드도 자동으로 final이 된다. 
- 다만 필드(변수)까지 final이 되는건 아니다.

final method: 메서드 수정 금지
- 클래스 전체는 상속이 돼도, 메서드만 변경할 수 없게 만들때 사용

과거에는 final -> inlining 으로 최적화 해줬다.
현대 (JIT 컴파일러) -> final 안붙여도 오버라이딩 안 했다 싶으면 알아서 인라인 최적화를 한다.
만약 나중에 오버라이딩 한 클래스가 로딩되면, 그때 다시 최적화를 취소함.

**Record, Enum** : 상속 불가능 (암묵적 final)


## Casting

객체의 참조 타입도 변환할 수 있다.

쓰임:
- `Employee` 변수에 담겨 있는 `Manager` 객체는 `setBonus()` 같은 매니저 고유의 메서드를 호출할 수 없음.    
- 이때 `(Manager)`로 캐스팅해서 다시 원래 타입으로 되돌려야 고유 기능을 쓸 수 있음.

upcasting(자식 -> 부모):
- 자동으로 됨
downcasting(부모 -> 자식):
- **명시적 캐스팅** `(Manager) staff[0]`이 필요함.

 `ClassCastException`
- 실제로는 일반 `Employee` 객체인데, 억지로 `(Manager)`로 캐스팅하려고 하면?
- 컴파일은 넘어가지만, 실행 시 `ClassCastException` 에러가 뜨고 프로그램이 종료됨.

`instanceof` 연산자
- 캐스팅 전에 진짜 그 타입이 맞는지 검사하는 것이 좋음.
- 문법: `if (staff[i] instanceof Manager) { ... }`
- 참고: `x`가 `null`이면 에러 없이 그냥 `false`를 반환함. (안전함)
	
**제약 사항**
- 상속 관계가 전혀 없는 클래스끼리(예: `String`으로) 캐스팅하려 하면 아예 **컴파일 에러**가 발생함.


캐스팅과 `instanceof`는 **최소한으로 사용하는 게 좋음**.
	
 `getSalary()` 처럼 다형성(동적 바인딩)으로 해결되는 건 그냥 두는 게 베스트.
	
 만약 캐스팅을 너무 자주 해야 한다면, "슈퍼클래스 설계가 잘못된 건 아닐까?"(예: `setBonus`를 부모에게 올려야 하나?) 고민해봐야 함.


## Pattern Matching for `instanceof`

Java 16부터 도입된 기능으로, `instanceof` 검사와 캐스팅을 한번에 처리해서 코드를 획기적으로 줄여줌.

Verbose
- 검사(`instanceof`) , 캐스팅(`(Manager)`) , 변수에 할당 --> class 세번 작성

Pattern Matching
- `if (obj instanceof Manager boss)`
- obj가 Manager 타입이 맞으면 즉시 Manager 타입으로 캐스팅되어 변수 boss에 할당된다.
- 만약 타입이 맞으면 if 블록 내부에서 `boss` 변수 사용가능. 
- 안맞으면, `boss` 생성 x, if문 실행 x

AND, OR
- `if (e instanceof Manager m && m.getBonus() >1000)`
- m이 참일때 뒤의 조건식에서 사용가능 (안전)
- `if (e instanceof Manager m && ...)`
- 앞부분이 거짓이어도 뒤의 조건식 테스트 해봐야함--> 에러

Shadowing
- `instanceof` 로 선언한 변수 이름이 클래스의 필드 이름과 같으면, 필드 변수가 가려짐


## Protected


같은 패키지 전체 && 다른 패키지의 자식 클래스가 상속 받을 수 있음.

접근 제한
- 다른 패키지에 있는 자식 클래스(`Administrator`)는 **자기 자신의** `protected` 필드는 건드릴 수 있음.
- 하지만 뜬금없이 생성한 **다른 부모 객체(`Employee`)의** `protected` 필드는 건드릴 수 없음.

ex)
```java
// 부모 클래스
package com.company.p1;

public class Employee {
	protected double salary; // protected field
}
```

```java
//자식 클래스
package com.company.p2;

import com.company.p1.Employee;

public class Manager extends Employee {
	public void method(){
		this.salary = 50000; // 상속받은 것. 접근 가능
		
		Employee e = new Employee();
		//e.salary = 50000; // 에러
		// 남의 부모 객체는 건드릴 수 없다. 자기가 상속받은 것에 대해서만 접근 가능.
	}
}
```