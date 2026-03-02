자바의 제네릭은 컴파일러의 기능이지 JVM의 기능이 아니다.
JVM 관점에서는 제네릭 타입의 객체라는 것이 존재하지 않는다. 모든 객체는 일반 클래스에 속한다.
제네릭은 기존의 제네릭이 없던 시절의 코드와 호환되도록 설계되었다. 초기 제네릭 구현은 제네릭 코드를 컴파일하여 초기 버전의 가상 머신에서도 실행할 수 있는 클래스 파일로 만들 수 있었다.
이것이 가능한 이유는 컴파일러가 타입 파라미터를 지워버리기 때문이다.

## Type Erasure

제네릭 타입을 정의하면, 컴파일러는 그에 상응하는 원시 타입을 자동으로 제공한다. 

Erasure 규칙
: 원시 타입의 이름은 제네릭 타입 이름에서 타입 파라미터를 제거한 것이다.
- T는 그 변수의 Bound 타입으로 교체된다. 별도의 제한이 없다면 `Object`로 교체된다.
ex) `Pair<T>`
```java
public class Pair<T> {
	private T first;
	private T second;
	// ...
}
```
- 컴파일 후 원시 타입
```java
public class Pair{
	private Object first;
	private Object second;
	
	public Pair(Object first, Object second){
		this.first = first;
		this.second = second;
	}
	
	public Object getFirst() {return first;}
	public void setFirst(Object newValue) {first = newValue;}
}
```
- 결과적으로, 이 코드는 제네릭이 도입되기 전에 작성했던 코드와 동일해진다. 

ex) 제한이 있는경우 `Interval<T>`
```java
public class Interval<T extends Comparable & Serializable> implements Serializable{
	private T lower;
	private T upper;
	
	public Interval(T first, T second){
		if(first.compareTo(second) <= 0){
			lower = first; upper=second;
		}
		else{
			lower = second; upper = first;
		}
	}
}
```
- 컴파일 후 원시 타입
```java
public class Interval implements Serializable{
	private Comparable lower; 
	private Comparable upper;
	
	public Interval(Comparable first, Comparable second){
		if(first.compareTo(second) <= 0){
			lower = first; upper = second'
		}
		// ...
	}
}
```
- 컴파일러는 첫 번째 제약 `Comparable`로 T 를 대체한다.
- 주의할 점은 만약 제한의 순서를 바꿔서 `<T extends Serializable & Comparable>`로 바꾼다면 T는 `Serializable`로 바뀐다. 문제는 `Serializable`인터페이스에는 `compareTo`메서드가 없다는 것이다. 따라서 컴파일러는 `compareTo`를 호출할 때마다 형변환 코드를 삽입해야한다. `((Comparable)first).compareTo(second)`. 그러므로 효율성을 위해, Tagging Interface(메서드가 없는 인터페이스)인 `Serializable`등의 제한은 제한 목록의 가장 끝에 두는 것이 좋다.


## Translating Generic Expressions

제네릭 메서드를 호출할 때, 리턴 타입이 소거되면 컴파일러는 자동으로 형변환 코드를 삽입한다.

예를 들어 `Pair<Employee>`를 보자
```java 
Pair<Employee> buddies = ...;
Employee buddy = buddies.getFirst();
```
- Erasure 후에 `getFirst()` 메서드는 `Object` 를 반환하도록 변경된다.
- 컴파일러는 위 코드를 두 단계의 가상 머신 명령어로 변환한다.
	- 1. 원시 메서드 `Pair.getFirst()`를 호출한다. (`Object` 타입)
	- 2. 반환된 `Object`를 `Employee`타입으로 형변환 한다.
만약 `Pair` 클래스의 필드가 `public` 이라면, 필드에 직접 접근할 때도 형변환이 삽입된다.
```java
Employee buddy = buddies.first;
```
- 이 코드 또한 바이트코드 수준에서는 `Object`를 가져온 후 `Employee`로 캐스팅하는 코드가 자동으로 추가된다.


## Translating Generic Methods

제네릭 메서드 역시 타입 소거 과정을 거친다. 하지만 이 과정에서 상속과 다형성이 깨질 위험이 발생한다. 이를 해결하기 위해 컴파일러는 Bridge Method 라는 특별한 메서드를 생성한다.

예를들어 `DateInterval`이라는 클래스가 `Pair<LocalDate>`를 상속받고, 두 날짜 중 두 번째 날짜가 첫 번째보다 앞서지 않도록 `setSecond` 를 오버라이딩한다고 생각해보자.
```java
class DateInterval extends Pair<LocalDate> {
	// override
	public void setSecond(LocalDate second){
		if(second.compareTo(getFirst()) >= 0)
			super.setSecond(second);
	}
}
```
여기서 문제가 발생한다.  타입 소거가 일어나면 부모와 자식 클래스의 메서드 시그니처가 달라지기 때문이다. 
- 부모 클래스 `Pair` : 소거 후 `setSecond(Object second)` 를 가진다
- 자식 클래스 `DateInterval` : `setSecond(LocalDate second)` 를 가진다.
이 두 메시드는 파라미터 타입이 다르므로, 자바 언어 규칙상 오버라이딩이 아니라 오버로딩이 된다. 따라서 다형성이 적용되지 않는다.
```java
Pair<localDate> pair = new DateInterval(...);
pair.setSecond(aDate);
```
`pair`는 `Pair` 타입이므로 `setSecond(Object)`를 호출할 것이고, `DateInterval`에는 `setSecond(Object)` 가 없으므로 부모의 메서드가 호출될 것이다.

이 문제를 해결하기 위해 컴파일러는 `DateInterval` 클래스 안에 브리지 메서드를 자동으로 넣는다
```java
public void setSecond(Object second){
	setSecond((LocalDate) second); //this.setSecond..
}
```
이 경우 `pair.setSecond(aDate)`를 호출하면, `pair`가 참조하는 객체는 `DateInterval`이므로, `DateInterval`의 `setSecond(Object)`가 호출됨 (브리지 메서드).
브리지 메서드가 값을 `LocalDate`로 캐스팅하여 `setSecond(LocalDate)`(우리가 작성한 메서드)를 호출. 결과적으로 다형성이 정상적으로 작동한다.


브리지 메서드는 리턴 타입이 다를 때도 생성된다. 
예를 들어 `getSecond`를 오버라이딩 하는 경우를 보자
```java
class DateInterval extends Pair<LocalDate> {
	public LocalDate getSecond() {
		return (LocalDate) super.getsecond();
	}
}
```
자바 소스코드에서는 파라미터가 같고 리턴 타입만 다른 두 메서드를 가질 수 없다. 하지만 자바 가상머신 내부에서는 리턴 타입도 메서드 시그니처의 일부로 취급하므로 이것이 가능한다.
따라서 컴파일러는 두 개의 메서드를 만든다. 
- `LocalDate getSecond()` : 사용자가 정의한 메서드
- `Object getSecond()` : 브리지 메서드. 내부에서 1번을 호출한다.



## Calling Legacy Code

Java 제네릭의 주요 설계 목표 중 하나는 제네릭 코드와 오래된 코드 간의 상호 운용성이다.

제네릭 객체를 레거시 메서드에 전달할 때를 보자
예를 들어 Swing의 `JSlider` 클래스를 보자. 이 클래스는 슬라이더의 tick에 라벨을 붙이는 기능을 제공한다.
- 레거시 코드(`JSlider`) : `setLabelTable` 메서드는 `Dictionary` 타입을 매개변수로 받는다. 하지만 `JSlider`는 제네릭으로 업데이트되지 않았기 때문에, 여전이 원시 타입인 `Dictionary`를 사용한다.
```java 
void setLabelTable(Dictionary table)
```
- 최신 코드 : 타입 안정성을 위해 제네릭을 사용하여 `Dictionary` 를 생성한다.
```java
Dictionary<Integer, Component> labelTable = new Hashtable<>();
labelTable.put(0, new JLabel(new ImageIcon("nine.gif")));
labelTable.put(20, new JLabel(new ImageIcon("ten.gif")));

slider.setLabelTable(labelTable);
```
- 여기서 컴파일러는 경고를 발생시킨다. `setLabelTable` 메서드가 원시 타입 `Dictionary`를 받아서, 그 안에 `Integer` 가 아닌 다른 타입을 넣을 수 있기 때문이다. 
- 그렇기에 `JSlider` 가 내부적으로 `labelTable`의 데이터를 읽기만 할 것이라는 점이 확실하다면, 이 경고는 무시해도 안전하다.

반대로 레거시 코드에서 원시 타입 객체를 가져와서 제네릭 변수에 담는 경우를 보자. 
```java
Dictionary<Integer, Component> labelTable = slider.getLabelTable(); 
```
- 컴파일러는 반환된 `Dictionary` 안에 정말로 `Integer`와 `Component` 만 들어있는지 보장할 수 없다. 그렇기에 경고가 있다. 

경고가 발생하는 이유를 이해했고 안전하다고 판단했다면, Annotation을 사용하여 경고 메시지가 뜨지 않게 할 수 있다.
- 지역 변수에 적용
```java
@SuppressWarnings("unchecked")
Dictionary<Integer, Component> labelTable = slider.getLabelTable();
```
- 메서드 전체에 적용
```java
@SuppressWarnings("unchecked)
public void configureSlider() {...}
```


