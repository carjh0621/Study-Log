
## Static Fields

If you define a field as static, then the field is not present in the objects of the class. There is only a single copy of each static field.
You can think of static fields as belonging to the class, not to the individual objects.

```java
class Employee
{
	private static int nextId = 1;
	private int id;
}
```
Every Employee object now has its own id field, but there is only one nextId field that is shared among all instances of the class.

There is a single static field nextId. Even if there are no Employee objects, the static field nextId is present.
사용 예시
```java
public Employee(String name){ // construct
	this.name =name;
	this.id = nextId; // 현재 공용 번호를 내 id로 설정
	nextId++; // 다음 사람을 위해 공용 번호 증가.
}
```


## Static Constants

Static variables are quite rare. However, static constants are more common

**대표 예시 
`Math.PI`**
- `public static final double PI = 3.14...;` 로 선언됨.
- - `Math.PI`로 바로 접근 가능.
- 만약 `static`이 없었다면? `Math` 객체를 생성해야만 접근 가능하고, 모든 `Math` 객체마다 PI 값을 따로 가져서 메모리 낭비가 심했을 것임.
**`System.out`:**
- 이것도 `public static final PrintStream out`으로 선언된 정적 상수

`System.setOut()` 메소드를 쓰면 `out`을 바꿀 수 있음


## Static Methods


Static methods are methods that do not operate on objects.
- `Math.pow(x, a)`

It does not use any `Math` object to carry out its task. In other words, it has no implicit parameter.

접근제한:
A static method of the Employee class cannot access the id instance field because it does not operate on an object. However, a static method can access a static field.

```java
public static int advanceId()
{
	int r= nextId;
	nextId++;
	return r;
}
...
int n = Employee.advanceId();
```

**호출 방식:**
- 권장: `ClassName.method()` (예: `Employee.advanceId()`)
- 비권장: `object.method()` (예: `harry.advanceId()`)
    - 문법상 에러는 아니지만, 마치 그 객체(`harry`)의 데이터를 쓰는 것처럼 오해를 줌. 코드 읽을 때 헷갈림.


사용 상황:
- **매개변수만 필요할 때:** 메서드 실행에 필요한 모든 데이터가 파라미터로 넘어올 때 (예: `Math.pow(x, a)` - `Math` 객체의 상태는 필요 없음).
- **정적 변수만 다룰 때:** 클래스의 `static` 필드만 조회하거나 변경할 때.



## Factory Methods

객체를 생성해주는 static methods. 
- `LocalDate.now()`
- `NumberFormat.getCurrencyInstance()`

사용 이유
1. **이름을 지을 수 있음:**    
    - 생성자는 이름이 무조건 클래스명과 같아야 함.
    - factory method 는 `getCurrencyInstance`(통화용), `getPercentInstance`(퍼센트용)처럼 이름만 봐도 어떤 용도의 객체인지 명확히 알 수 있음.
        
2. **리턴 타입이 유연함:**
    - 생성자는 무조건 그 클래스 자체의 인스턴스만 생성함.
    - 팩토리 메서드는 해당 클래스의 **자식 클래스(Subclass)** 객체를 반환할 수도 있음. (예: `NumberFormat`의 메서드는 실제로는 자식인 `DecimalFormat` 객체를 리턴함).


## Main Method

**왜 static인가?:** 프로그램이 시작되는 시점에는 메모리에 생성된 객체가 하나도 없음. 객체가 없어도 실행돼야 하므로 `main`은 반드시 `static`이어야 함.

- **단위 테스트(Unit Test):**
    - 모든 클래스는 자신만의 `main` 메서드를 가질 수 있음.
    - 개발 중에 `Employee` 클래스가 잘 돌아가는지 확인하려고 `Employee` 안에 `main`을 만들어서 테스트 코드를 넣을 수 있음.
	- **실행 방법:**
		- `java Application`을 실행하면 `Application` 클래스의 `main`만 실행되고, `Employee` 안에 있는 `main`은 무시됨.
		- `java Employee`를 실행하면 `Employee` 클래스의 `main`이 실행됨 (단독 테스트 가능).

