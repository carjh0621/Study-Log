
제네릭 프로그래밍은 다양한 타입의 객체들을 재사용할 수 있는 코드를 작성하는 기법이다.

예를들어 `String` 객체를 모으기 위한 클래스와 `File` 객체를 모으기 위한 클래스르 각가  따로 만들 필요가 없다. `ArrayList` 라는 단일 클래스 하나만으로 어떤 클래스의 객체든 수집할 수 있다.
Java는 제네릭 클래스가 도입되기 전부터 `ArrayList` 클래스를 가지고 있었다. 제네릭 메커니즘이 어떻게 진화했는지 알아보자

## The Advantage of Type Parameters

제네릭 도입 이전과 이후를 비교해보자

제네릭 도입 전 (상속)
: `ArrayList` 클래스는 모든 클래스의 상위 클래스인 `Object` 참조의 배열을 유지하는 방식으로 구현되었다.
```java
public class ArrayList 
{
	private Object[] elementDate; // 모든 객체를 담을 수 있는 Object 배열.
	//...
	public Object get(int i) {...}
	public void add(Object o) {...}
}
```
- 여기에는 두가지 문제점이 있다.
- 1. `get` 메서드가 `Object` 타입을 반환하기 때문에, 값을 꺼낼 때마다 반드시 해당 타입으로 Casting을 해야한다.
- 2. 컴파일러 차원에서 타입 검사가 이루어지지 않는다. `String` 을 의도한 리스트에 실수로 `File` 객체를 넣어도 컴파일 및 실행이 문제없이 진행된다. 
	- 하지만 나중에 이 값을 `get`으로 꺼내어 `String`으로 캐스팅 하려고 할 때, ClassCastException이 발생한다.

제네릭 도입 후
: 제네릭은 타입 파라미터를 통해 더 나은 방식을 제공한다. `ArrayList` 클래스는 이제 요소의 타입을 명시하는 타입 파라미터를 가진다.
- ex) `var files = new ArrayList<String>();`
- 이 코드는 `files` 라는 리스트가 `String` 객체만 담는다는 것을 보여주므로 가독성이 좋아진다.
: Notes
- 다이아몬드 구문 (Diamond Syntax)
	- 변수를 선언할 때 타입을 명시했다면, 생성자에서는 타입 파라미터를 생략하고 `<>`만 사용할 수 있다. 타입은 변수 타입을 통해 추론된다.
	- `ArrayList<String> files = new ArrayList<>();`
- Java 9 부터는 익명 서브클래스에서도 다이아몬드 구문을 사용할 수 있게 되었다.
	```java 
	ArrayList<String> passwords = new ArrayList<>()
	{
		// 오버라이딩
		public String get(int n){
			return super.get(n).replaceAll(".","*");
		}
	}
	```

: 컴파일러의 역할
- 제네릭을 사용하면 컴파일러가 타입 정보를 활용할 수 있게 되어 코드가 훨씬 안전해진다.
- 1. `get`메서드 호출 시 별도의 캐스팅이 필요 없어진다. 컴파일러는 반환 타입이 `Object`가 아니라 `String`임을 이미 알고 있기에
- 2. `add` 메서드 또한 `string` 타입의 파라미터를 갖는다는 사실을 컴파일러가 인지한다. 따라서 잘못된 타입의 객체를 삽입하려 하면 컴파일 에러가 발생한다.
- --> 런타입에 발생하는 오류가 아닌 컴파일 시점에 발생하는 에러가 더 쉽게 수정 가능하기 때문에 프로그램이 더 읽기 쉬워지고 안전해진다.


## Who Wants to Be a Generic Programmer?

Generic은 사용하는 사람에게는 편리하지만, 라이브러리 설계자는 고민을 많이 해야한다.
대부분의 Java 이용자들은 `ArrayList<String>` 과 같은 제네릭 클래스를 마치 언어에 내장된 기능인 `String[]`배열처럼 쉽게 사용한다.
하지만 만드는 사람은 다음과 같은 사항들을 고려해야한다.
- 사용자가 타입 파라미터에 어떤 클래스를 집어넣더라도 작동해야한다.
- 사용자에게성가진 제약을 주어서는 안된다.
- 오류 메시지가 난해하지 않아야한다.
- 미래에 이 클래스가 어떻게 사용될지 모든 가능성을 예측해야한다.

제네릭 설계의 어려움
: `addAll` 메서드 예시를 통해 어떤 점이 어려운지 살펴보자.
- ex) `Manager` 클래스가 `Employee`클래스르 상속받았다고 가정
```java
class Employee{...}
class Manager extends Employee {...}

ArrayList<Employee> staff = new ArrayList<>();
ArrayList<Manager> bosses = new ArrayList<>();
```
- 문제상황: `ArrayList`에는 다른 컬렉션의 요소를 모두 추가하는 `addAll` 메서드가 있다.
	- 직원 명단 `staff` 에 매니저 명단 `bosses` 를 추가하는 것은 가능해야한다.
		- `staff.addAll(bosses);`
	- 하지만 그 반대의 경우는 막아야 한다.
- 해결책 : Wildcard type (` extends E` 등) 

제네릭 타입 숙련도
- 1. Basic Level
    - 제네릭이 어떻게 또는 왜 작동하는지 깊게 고민하지 않고 `ArrayList` 같은 제네릭 클래스를 단순히 사용하는 단계
- 2. Troubleshooting Level)
    - 서로 다른 제네릭 클래스를 섞어서 사용하거나, 타입 파라미터를 모르는 레거시 코드(Legacy Code)와 연동할 때 혼란스러운 에러 메시지를 마주하게 된다.
    - 이때는 무작위로 코드를 고쳐보는(tinkering) 것이 아니라, 문제를 체계적으로 해결하기 위해 Java 제네릭의 작동 원리를 배워야 한다
- 3. Implementation Level
    - 자신만의 제네릭 클래스나 메서드를 직접 구현하는 단계

언제 제네릭을 직접 구현해야 하느나?
: 일반적으로 `Object`나 `Comparable` 인터페이스 같은 매우 일반적인 타입으로 캐스팅을 자주해야하는 코드를 작성할 때만 제네릭 타입 파라미터의 이점을 얻을 수 있다.


