
## Interface concept

인터페이스는 클래스가 아니다. 
이는 어떤 클래스가 준수해야할 요구사항의 집합이다.

ex) `Arrays.sort` 메서드의 경우 객체들의 배열을 정렬해주지만, 배열안의 객체들이 `Comparable` 인터페이스를 구현해야 한다는 조건이 있다. 

특징
- 인터페이스의 모든 메서드는 자동으로 `public` 이자 `abstract`(구현부 x) (java 8 이전 기준)
- 인터페이스는 인스턴스 필드를 가질 수 없다.
- 클래스는 `implements` 키워드를 통해 인터페이스를 구현한다고 선언

`Commparable` 인터페이스 구조
- generic 적용 (java 5 이후)
```java
public interface Comparable<T>{
	int compareTo(T other);
}
```
- 타입 파라미터 T --> 컴파일 시점에 타입을 체크할 수 있음.
- `Employee` 클래스면 `Comparable<Employee>` 를 구현하여 캐스팅 없이 바로 `Employee` 객체를 인자로 받을 수 있다.

`CompareTo` 메서드
- `x.compareTo(y)` 를 호출했을 때 반환값의 의미
	- 음수 : `x < y`
	- 0 : `x = y`
	- 양수 : `x > y`


인터페이스 구현
- 두 단계가 필요
	- 선언: 클래스 선언부에 `implements InterfaceName`을 추가
	- 정의: 인터페이스에 포함된 모든 메서드를 구현?
- 주의사항
	- 인터페이스 내부의 메서드는 명시하지 않아도 `public`
	- 다만, 이를 구현하는 클래스에서는 반드시 `public` 접근 제어자를 명시해야한다.
	- 만약 `public`을 생략하면, 자바 컴파일러는 이를 패키지전용 접근 권한으로 간주
	- 이는 인터페이스(public)보다 더 좁은 권한을 할당하려는 시도 --> 컴파일 에러

`CompareTo` 구현
- 정수형 비교
	- 단순 뺄셈(`this.id - other.id`)을 사용할 수 있으나, 오버플로우 위험이 있다.
	- 안전한 방법: `Integer.compare(x, y)` 정적 메서드 사용 권장.
	- 뺄셈 사용 조건: 값의 범위가 충분히 작아서 오버플로우가 발생하지 않음이 확실할 때만 사용.
- 실수형 비교
	- 뺄셈(`salary - other.salary`)을 사용하면 소수점 오차로 인해 값이 달라도 0으로 반올림될 위험이 있다.
	- 해결책: `Double.compare(x, y)` 사용. (x < y면 -1, x > y면 1 반환)
- equals 메서드
	- `compareTo`의 결과가 0이라면, `equals`의 결과도 `true`여야 하는 것이 이상적
	- 예외: `BigDecimal` 클래스. `1.0`과 `1.00`은 `equals`로는 `false`(정밀도 차이)지만, `compareTo`로는 0(값 동일).


상속과 CompareTo
- 상속 관계에서 비교 로직을 작성할 때 대칭성 문제가 발생할 수 있다.
- 문제 상황 (`Manager extends Employee)
	- `Manager`가 `CompareTo`를 오버라이드 하여 `Employee`와 비교하려 할 때, `Employee`객체를 `Manager` 로 캐스팅 하면 `ClassCastException`이 발생 가능
- 해결책
	- 하위 클래스마다 비교 방식이 다른 경우
		- 서로다른 클래스끼리의 비교를 원천 차단
		- `if (getClass() != other.getClass()) throw new ClassCastException();`
	- 상위 클래스의 비교 로직을 공통으로 사용하는 경우
		- 상위 클래스의 `compareTo` 메서드를 `final` 로 선언하여 하위 클래스에서 변경하지못하게 한다.

ex)
```java
public class Employee implements Comparable<Employee> {
	...
	
	@Override
	public int compareto(Employee other){
		return Double.compare(salary, other.salary);
	}
}
```


---
자바의 정렬 알고리즘은 오름차순 정렬을 기본 목표로 설계되어 있다. 그래서 `compareTo`의 반환값을 보고 다음과 같이 행동한다.

| 반환값    | 의미 (판사의 판결)          | 자바(sort 메서드)의 행동 | 결과 (오름차순 기준)     |
| ------ | -------------------- | ---------------- | ---------------- |
| 양수 (+) | `this`가 `other`보다 크다 | 자리를 바꾼다          | 큰 녀석을 뒤로 보냄      |
| 음수 (-) | `this`가 `other`보다 작다 | 그대로 둔다           | 작은 녀석이 앞에 있으니 OK |
| 0      | 둘이 같다                | 그대로 둔다           | 바꿀 필요 없음         |
- `Comparable` 인터페이스 문서에는 이렇게 적혀 있다. "이 객체가 주어진 객체보다 작으면 음의 정수, 같으면 0, 크면 양의 정수를 반환해야 한다."

- 만약 반대로 내림차순으로 정렬하고 싶다면?
```java
public int compareTo(Employee other) {
    // 내림차순 팁: 파라미터 순서를 바꿔서 비교하거나, 결과에 -1을 곱함
    return Double.compare(other.salary, this.salary); 
}
```

---



## Properties of Interfaces

인터페이스는 클래스가 아니므로 `new` 연산자를 사용하여 객체를 직접 생성할 수 없다.

객체는 만들 수는 없지만, 인터페이스 타입의 변수는 선언할 수 있다. 이 변수는 해당 인터페이스를 구현한 클래스의 객체를 참조할 수 있다.
```java
Comparable x;
x = new Employee(...); //Employee 가 Comparable을 구현했다면 할당 가능하다.
```

`instanceof` 연산자를 사용하여 어떤 객체가 특정 인터페이스를 구현했는지 확인할 수 있다.
```java
if (anObject instanceof Comparable) { ... }
```

인터페이스의 상속
- 인터페이스도 클래스처럼 상속이 가능하다. 
- `extends` 키워드 사용
ex)
```java
// 기본 인터페이스
public interface Moveable {
	void move(double x, double y);
}

//확장된 인터페이스
public interface Powered extends Moveable{
	double milesPerGallon();
}
```


인터페이스 필드
- 상수는 정의할 수 있다.
- 인터페이스 내부에 정의된 모든 필드는 자동으로 `public static final` 이 된다. (명시하지 않아도 됨)
ex)
```java
public interface Powered extends Moveable {
    double milesPerGallon(); 
    double SPEED_LIMIT = 95; // public static final이 생략된 상수
}
```

multiple implementation
- 클래스 상속은 오직 하나만 가능. 
- 인터페이스 구현은 동시에 여러 개를 할 수 있다. 
- 이는 자바에서 클래스의 동작을 유연하게 정의하는 방법
```java
class Employee implements Cloneable, Comparable { ... }
```


레코드(Records)와 열거형(Enum)
- `record`나 `enum`은 암묵적으로 각각 `Record`, `Enum` 클래스를 상속받기 때문에 다른 클래스를 상속할 수 없다.
- 하지만 인터페이스를 `implements`하는 것은 가능
    

봉인된 인터페이스 (Sealed Interfaces)
- 최신 자바 버전에서는 클래스와 마찬가지로 인터페이스도 `sealed`로 선언할 수 있다.
- `permits` 절을 통해 이 인터페이스를 구현하거나 확장할 수 있는 클래스/인터페이스를 엄격하게 제한할 수 있음.

---
인터페이스: 이런 기능을 반드시 제공해야 한다는 규격서
클래스 : 그 규격을 실제로 만족하는 구현체

- `Moveable`은 “움직일 수 있어야 해”라는 **규약(타입)**
- `Car`는 “진짜 자동차 구현(필드, 로직)” + “Moveable/Powered 규약 준수”
ex)
```java
public interface Moveable {
    void move(double x, double y);
}
...
public class Car implements Moveable {
    private double x, y;

    @Override
    public void move(double x, double y) {
        this.x += x;
        this.y += y;
    }
}
// 이제 Car 는 Moveable로 취급 가능해진다.
...
Moveable m = new Car();     // 변수 타입은 인터페이스
m.move(3, 4);               // 실행은 Car의 move가 됨(동적 바인딩)
```

---



## Interfaces and Abstract Classes

`Comparable`을 추상클래스로?
```java
abstract class Comparable { // 인터페이스가 아니라 추상 클래스라고 가정
    public abstract int compareTo(Object other);
}

// Employee가 이를 상속받아 구현
class Employee extends Comparable {
    public int compareTo(Object other) { ... }
}
```

문제점
- 자바는 클래스에 대해 단일 상속만을 허용. 
```java
class Employee extends Person, Comparable // 에러 (ERROR): 다중 상속 불가
```

예시: `CharSequence` 인터페이스 사용
- `CharSequence`는 문자들의 순서(문자열처럼 보이는 것)를 표현하는 인터페이스
- 문자들을 읽을 수 있는 최소 기능만 정의해 둔 규약
- 대표 메서드:
	- `int length()` : 길이
	- `char charAt(int index)` : i번째 문자
	- `CharSequence subSequence(int start, int end)` : 부분 구간
	- `boolean isEmpty()` : 비었는지 (Java 17+)

- 때로는 String 대신 CharSequence로 받는게 유용하다.
- 예를 들어 `public void process(String s) { ... }` 와같이 매개변수의 타입이 String 인 경우
	- 이 함수는 오직 String 만 받을 수 있다.
	- 하지만 실제로는 문자열 데이터가 꼭 String 으로만 오지 않는다. (StringBuilder, StringBuffer, CharBuffer 등)
- 이럴 때 `public void process(CharSequence s) { ... }` 처럼 `CharSequence` 로 매개변수를 받으면,
	- String, StringBuilder, StringBuffer 등을 다 받을 수 있다. 
	- 왜냐하면 모두 CharSequence 인터페이스를 구현하기 때문이다.
- 다만, `CharSequence` 는 최소한의 읽기 기능만 제공한다. 


## Static and Private Methods

자바 8 이전에는 인터페이스에 정적 메서드를 넣을 수 없었다. 그래서 개발자들은 인터페이스와 짝을 이루는 Companion class 를 별도로 만들어야 했다.

자바8 부터는 인터페이스 안에 `static` 메서드를 직접 정의할 수 있게 되었다.

ex)
java 11 이전: 별도의 `Paths` 사용
```java
Path p = Paths.get("jdk-17", "conf", "security");
```
java 11 이후 권장: `Path` 인터페이스의 정적 메서드 사용
```java
public interface Path {
    // 인터페이스 내부에 정적 팩토리 메서드 포함
    public static Path of(String first, String... more) { ... }
}

// 사용 코드
Path p = Path.of("jdk-17", "conf", "security");
```


private method
- 자바 9부터는 인터페이스 내부에 `private` 메서드를 정의할 수 있음.
	- private 메서드는 정적일수도 있고 일반적인 메서드일 수 있다.
	- 인터페이스 외부에서는 호출할 수 없으며, 오직 인터페이스 내부의 다른 메서드에서만 호출 가능


## Default Methods

인터페이스 내부에 body를 가진 메서드를 정의할 수 있는 기능
- `default` 를 붙여야 한다.
- 해당 인터페이스를 구현하는 클래스에서 이 메서드를 오버라이드 하지 않으면, 인터페이스에 정의된 기본 코드가 실행된다.
ex)
```java
public interface Comparable<T> {
    default int compareTo(T other) { return 0; }
}
```

사용 하는 이유
- 1. 인터페이스에 메서드가 많지만, 구현체에서 모든 기능이 필요하지 않을 때
- ex) `Iterator` 인터페이스
	- `hasnext(), next(), remove()`
	- read-only iterator를 만들더라도 `remove()`를 빈 껍데기로라도 구현해야 함.
	- 현재, 인터페이스에서 `UnsupportedOperationException`을 던지는 default메서드로 만들어두면, 구현 크래스는 `remove()`를 신경쓰지 않아도 된다. 
```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    
    // 구현 클래스가 오버라이드 안 하면 자동으로 예외 발생
    default void remove() { 
        throw new UnsupportedOperationException("remove"); 
    }
}
```
- 2. 다른 추상 메서드를 활용하여 기능을 완성할 수 있을때
- ex) `Collection` 인터페이스
	- `size(), isEmpty()`
	- `isEmpty()` 는 `size() == 0` 과 같으므로, default method로 미리 구현할 수 있다. 
```java
public interface Collection {
    int size(); // 추상 메서드
    
    // size()를 활용한 디폴트 구현
    default boolean isEmpty() { 
        return size() == 0; 
    }
}
```

default method가 도입된 이유
- 기존 코드를 유지한 채로 인터페이스에 새로운 기능을 추가하기 위해서
- ex) `Collection` 인터페이스와 이를 구현한 `Bag` 클래스가 존재한다고 가정
	- 이때 `Collection` 인터페이스에 `stream()` 메서드를 추가한다고 하자.
- 이런 상황에서는 다음과 같은 문제들이 발생한다.
	- `Bag` 클래스를 다시 컴파일하려고 하면 에러 발생. --> 새로 추가된 `stream` 메서드를 구현하지 않았기 때문에. (Source Compatibility 가 깨진 상황)
	- `Bag` 클래스를 재컴파일하지 않고 예전 JAR 파일 그대로 실행은 가능하다. 다만, 어디선가 `bagInstance.stream()` 을 호출하는 순간, JVM은 `AbstractMethodError`을 일으킨다. (Binary Compatibility 문제)
- --> `stream`을 default method 로 선언하면 두 문제가 모두 해결된다.
	- `stream`을 구현하지 않아도, 인터페이스의 디폴트 구현을 사용하므로 컴파일된다
	- 재컴파일 없이 JAR 파일을 재실행해서 기존 `Bag` 클래스를 로드해서 `stream()` 을 호출하더라도, `Collection.stream`의 default method가 실행돼서 에러가 나지 않는다. 

	*상황은 아마도 Collection 인터페이스는 (JDK/라이브러리 제작자가) 새 버전으로 컴파일해서 배포한 상태, 실행시점에 그 새로운 Collectioin.class를 로드하고 Bag.class는 예전 jar 그대로 로드하는 상황.


## Resolving Default Method Conflicts

자바에서 동일한 시그니처(이름, 파라미터)를 가진 메서드가 여러 인터페이스나 슈퍼클래스에 동시에 존재할 때, 충돌을 해결하는 규칙은 다음과 같다
1. Superclasses Win
	- 슈퍼 클래스에 구체적인 메서드가 존재한다면, 인터페이스의 디폴트 메서드는 무시된다.
2. Interfaces Clash
	- 두 개 이상의 인터페이스가 같은 메서드를 가지고 있고, 그 중 최소 하나가 default 메서드라면 컴파일러는 이를 충돌로 간주한다.
	- --> 개발자가 해당 클래스에서 메서드를 오버라이드하여 명시적으로 충돌을 해결해야 한다.

ex) 두 인터페이스에서 동일한 메서드를 가지고 있을 때
```java
interface Person {
    default String getName() { return ""; }
}

interface Named {
    default String getName() { return getClass().getName() + "_" + hashCode(); }
}

// 컴파일 에러 (Error): Person과 Named의 getName() 충돌
class Student implements Person, Named { ... }

// 해결책: 클래스에서메서드를 오버라이드 하여 어느 쪽 로직을 따를지 명시
// InterfaceName.super.method() 사용 
class Student implements Person, Named {
    @Override
    public String getName() {
        // Person의 디폴트 메서드를 명시적으로 호출
        return Person.super.getName();
    }
}
```


주의사항
- Object 클래스의 메서드 (`toString, equals, hashCode`) 는 default 메서드로 재정의 할 수 없다.
- 결국 인터페이스의 `toString`은 절대 호출될 일이 없으므로, 자바는 이를 아예 컴파일 에러로 막아버린다.


## Interfaces and Callbacks

Callback
- 콜백 패턴은 특정 이벤트가 발생하면, 이 코드를 실행하라고 시스템에 요청하는 프로그래밍 기법이다.
- ex) 1초가 지날 때마다, 시계를 업데이트해줘

c언어 등에서는 함수 포인터를 사용하여 함수의 주소를 넘기지만, 자바는 객체를 넘기는 방식을 사용.
- 실행하고 싶은 메서드가 포함된 객체를 타이머에게 전달
- 타이머는 전달받은 객체가 어떤 메서드를 가지고 있는지 확신할 수 있어야 한다. 이를 위해서 `ActionListener` 같은 인터페이스를 사용
- 단순히 함수만 전달하는 것보다 객체를 전달하면, 객체 내부에 추가적인 정보를 함께 전달할 수 있어서 더 유연하다.

javax.swing.Timer
- 정해진 시간 간격마다 이벤트를 발생시키는 클래스
- `java.util.Timer`와는 다르다.

ActionListener 인터페이스
- `java.awt.event` 패키지에 존재. 타이머가 호출할 메서드를 정의하고 있는 규약
```java
public interface ActionListener{
	void actionPerformed(ActionEvent event);
}
```

ActionEvent
- 이벤트에 대한 정보를 담고 있는 객체. (이벤트가 발생한 시간 등)

구현
- `ActionListener` 인텉페이스를 구현하는 클래스를 만든다
- `actionPerformed` 메서드 안에 주기적으로 실행될 코드를 작성
- 위에서 만든 클래스의 인스턴스를 생성
- `Timer` 객체를 생성할 때 시간 간격과 listener 객체를 전달한다.
- `timer.start()`를 호출한다.
ex)
```java
// ActionListener 인터페이스를 구현하는 클래스  
class TimePrinter implements ActionListener {  
  
    // 인터페이스에 정의된 메서드 구현  
    @Override  
    public void actionPerformed(ActionEvent event) {  
        // event.getWhen(): 이벤트 발생 시간(long, epoch milliseconds)  
        // Instant.ofEpochMilli(): 사람이 읽기 쉬운 시간 객체로 변환  
        System.out.println("At the tone, the time is "  
                + Instant.ofEpochMilli(event.getWhen()));  
  
        // 시스템 비프음 발생  
        Toolkit.getDefaultToolkit().beep();  
    }  
}
...
var listener = new TimePrinter();
var timer = new Timer(1000, listener);
timer.start();
```

---
Timer의 두 번째 파라미터 타입은 `ActionListener` 인터페이스.

---


## Comparator Interface

`Comparable` 인터페이스는 클래스 내부에 구현되어 Natural Ordering 을 정의한다. 하지만 다음과 같은 상황에서는 이것만으로 부족하다.
- 이미 정의된 정렬 기준 말고 다른 기준으로 정렬하고 싶을 때. (사전순 -> 글자 길이순)
- `String` 클래스처럼 내가 만들지 않은 클래스라서 소스 코드를 수정해서 `Comparable`을 구현하거나 변경할 수 없을 때.

| 특징  | Comparable                     | Comparator (외부 기준)               |
| --- | ------------------------------ | -------------------------------- |
| 위치  | 정렬하려는 객체 클래스 내부 (`implements`) | 별도의 클래스 생성                       |
| 메서드 | `int compareTo(T other)`       | `int compare(T first, T second)` |
| 관점  | 본인과 비교                         | A와 B 중 누가 큰지                     |
| 활용  | `Arrays.sort(arr)`             | `Arrays.sort(arr, comparator)`   |
ex) `String`을 길이순으로 정렬하는 Comparator
```java
import java.util.Comparator;

class LengthComparator implements Comparator<String> {
    @Override
    public int compare(String first, String second) {
        // 첫 번째 문자열의 길이가 길면 양수, 짧으면 음수 리턴
        return first.length() - second.length();
    }
}
//compare 메서드도 compareTo와 동일한 반환값 규칙(음수, 0, 양수)을 따른다 
//길이 값은 항상 0 이상이고 차이가 크지 않으므로 단순히 뺄셈을 해도 안전.
...
...
String[] friends = { "Peter", "Paul", "Mary" };

// 단순히 sort만 부르면 String의 기본인 '사전순' 정렬이 됨
// Arrays.sort(friends); 

// Comparator 인스턴스(new LengthComparator())를 함께 전달
Arrays.sort(friends, new LengthComparator());
```



## Object Cloning

copy의 종류
- reference copy
	- `Employee original = new Employee("John", 50000);`
	- `Employee copy = original;`
	- original 과 copy는 동일한 메모리 주소를 가리킨다. copy를 수정하면 original 도 바뀐다
- Object cloning
	- `Employee copy = original.clone();`
	- original 과 동일한 상태를 가진 새로운 객체를 만든다. copy를 수정해도 original 은 영향을 받지 않는다.

Object 클래스의 clone() 메서드는 기본적으로 앝은 복사를 수행한다.
- 즉, 필드별로 값을 그대로 복사함.
	- int, double 등은 값이 복사
	- String 등은 어차피 내용을 바꿀 수 없으므로 공유해도 안전
	- Date, 배열 등의 가변 객체는 객체의 주소값만 복사되므로, 서로 영향을 미치게 된다.

Deep Copy 구현
- 가변 필드(Mutable Fields)가 포함된 클래스를 안전하게 복사하려면 깊은복사를 구현해야한다.
- `clone()` 내부에서 `super.clone()` 을 호출해서 껍데기를 만든 후 , 가변 필드들을 일일이 새로 복제해서 넣어야 한다.

Cloneable 인터페이스 구현 규칙
- Cloneable 인터페이스 구현 (메서드 x = Tagging Interface (복제 허용 표식?))
- `Object.clone()` 은 protected. 이를 public으로 오버라이드 해야한다.
- `CloneNotSupportedException` 을 throw, try-catch 해야한다. 
- java 5 부터는 오버라이드 할 때 리턴 타입을 `Object` 대신 자신의 클래스 타입으로 좁혀서 정의할 수 있다. --> 캐스팅 불필요

ex)
```java
public class Employee implements Cloneable{
	...
	@Override
	public Employee clone() throws CloneNotSupportedException{
		Employee cloned = (Employee) super.clone();
		cloned.hireDay = (Date) this.hireDay.clone(); 
		return cloned;
	}
}
```



