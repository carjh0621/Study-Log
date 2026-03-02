대부분의 제약 사항은 Type Erasure 때문에 발생한다.
## Type Parameters Cannot Be Instantiated with Primitive Types

제네릭 타입 파라미터에는 기본 타입을 사용할 수 없다.
타입 소거 후에 `Pair` 클래스 내부의 필드는 `Object` 타입이 된다. `Object`는 객체의 참조만 담을 수 있고 `double`같은 기본 값 자체를 담을 수는 없기 때문이다. 
이는 자바 언어에서 기본 타입과 객체 타입이 분리되어 있어서 생기는 일이지만, Wrapper Class를 사용하면 해결된다.

## Runtime Type Inquiry Only Works with Raw Types

JVM에 있는 객체들은 항상 일반 클래스이다. 따라서 런타임에 타입을 조화하면 항상 제네릭 정보가 지워진 원시 타입만 확인할 수 있다.

`instanceof` 사용 불가
: `instanceof`는 런타임에 타입을 검사하는 연산자이다. 런타임에는 `<String>`과 같은 정보가 없으므로 다음 코드는 에러가 발생한다.
```java
if (a instanceof Pair<String>) // 컴파일 에러 (ERROR)
if (a instanceof Pair<T>)      // 컴파일 에러 (ERROR)
```
- 오직 원시 타입인 `Pair` 인지 아닌지만 검사할 수 있다.

Casting 경고
: 강제 형 변환을 시도하면 경고가 발생한다.
```java
Pair<String> p = (Pair<String>) a; // 경고 (Warning)
```
컴파일러가 런타임에 `Pair<String>` 인지 확인할 방법이 없음을 알려주는 것이다.

`getClass()` 메서드
: `getClass()`는 항상 원시 타입을 반환한다.
```java
Pair<String> stringPair = ...;
Pair<Employee> employeePair = ...;

if (stringPair.getClass() == employeePair.getClass()) // true 반환
```
- 둘 다 런타임에는 똑같은 `Pair.class`이기때문에 결과는 true이다. 


## You Cannot Create Arrays of Parameterized Types

`new Pair<String>[10]` 과 같이 제네릭 타입의 배열을 생성하는 것은 금지되어 있다.
```java
var table = new Pair<String>[10]; // 컴파일 에러
```
변수 선언은 가능하지만(`Pair<String>[] table;`) 초기화는 불가능하다

이유는 배열은 런타임에 자신의 요소 타입을 알고 확인한다. 하지만 제네릭은 타입이 소거된다. 이 두가지 특성이 충돌하면 타입 안정성이 깨진다.



## Varargs Warnings

위에서 제네릭 타입의 배열 생성은 불가능하다고 했다. 하지만 가변 인자(Varargs) 메서드를 사용할 때 이 규칙과 충돌이 발생한다.

자바에서 가변 인자 (`T... ts`) 는 내부적으로 배열로 처리된다. 제네릭 타입의 가변 인자를 받는 메서드를 보면 다음과 같다
```java
public static <T> void addAll(Collection<T> coll, T... ts)
{
    for (T t : ts) coll.add(t);
}
```
이제 이 메서드를 호출할 때 제네릭 타입을 전달한다고 해보자
```java
Collection<Pair<String>> table = ...;
Pair<String> pair1 = ...;
Pair<String> pair2 = ...;

addAll(table, pair1, pair2); //문제
```
- JVM은 `pair1` `pair2` 를 담기 위해 내부적으로배열을 생성해야 한다.
- 이 배열은 `Pair<String>[]` 이어야 한다. 이는 제네릭 배열 생성 금지 규칙에 위배된다.
- 하지만 자바는 사용 편의를 위해 이 경우 규칙을 완화하여 에러 대신 경고만 발생시킨다.

이 경고를 없애는 방법은 두가지가 있다
- 1. 호출하는 메서드에 `@SuppressWarnings("unchecked")`를 붙인다
- 2. `addAll` 메서드 자체에 `@SafeVarargs` annotation을 붙인다.
```java
@SafeVarargs 
public static <T> void allAll(Collection<T> coll, T... ts)
```
주의할 점은 이 annotation은 오버라이딩될 가능성이 없는 메서드에만 사용할 수 있다. (`static`,`final`,`private`). 오버라이딩 가능한 메서드는 하위 클래스에서안전하지 않은 동작을 할 수 있기 때문에 사용이 금지된다. 

Hidden Danger
:`@SafeVarargs`를 이용해 제네릭 배열 생성 금지를 우회하여 배열을 반환하는 메서드를 만들 수도 있다
```java
@SafeVarargs static <E> E[] array(E... array) {return array;}
```
하지만 이는 힙 오염을 유발할 수 있다. 
- 반환된 배열의 실제 런타임 타입은 소거에 의해 `Object[]` 이다.
- 이를 `Object[]` 변수에 담아 다른 타입의 객체를 저장해도 `ArrayStoreException`이 발생하지 않는다.
- 나중에 이 배열에서 값을 꺼내려 할 때 런타임 에러가 터질 수 있다.


## You Cannot Instantiate Type Variables

제네릭 코드 내부에서 `new T(...)` 와 같은 표현식은 사용할 수 없다.
```java
public Pair() { first = new T(); second = new T();} //컴파일 에러
```
- 타입이 소거되면 T는 `Object`로 변한다. 개발자가 의도한 것은 특정 타입의 객체 생성이지, `new Object()`는 아니기 때문이다

해결책은 두가지가 있다.
- 1. Java 8 이후에 사용가능한 가장 좋은 방법은 호출자가 생성자 참조를 전달하게 하는 것이다.
```java
// String 생성자를 전달
Pair<String> p = Pair.makePair(String::new);

public static <T> Pair<T> makePair(Supplier<T> constr)
{
	return new Pair<>(constr.get(), constr.get());
}
```
- 2. Reflection 사용
	- Java8 이전이나 reflection을 꼭 써야 할 때는 `Class` 객체를 전달받아야 한다.
	- `T.class`는 소거되면 `Object.class`가 되므로 사용할 수 없다.
	- 따라서 `Class<T>`를 인자로 받도록 설계한다.
```java
Pair<String> p = Pair.makePair(String.class);

public static <T> Pair<T> makePair(Class<T> cl){
	try{
		return new Pair<>(cl.getConstructor().newInstance(), 
							cl.getConstructor().newInstance());
	}
	catch(Exception e) {return null;}
}
```



## You Cannot Construct a Generic Array

제네릭 타입의 단일 인스턴스를 만들 수 없는 것처럼, 제네릭 타입의 배열도 생성할 수 없다. 

배열은 런타임에 자신이 담을 수 있는 타입 정보를 가지고 있어야 한다. 하지만 제네릭은 컴파일 시에 타입이 소거된다.
```java
public static <T extends Comparable> T[] minmax(T... a)
{
	T[] mm = new T[2]; //컴파일에러
	// ... 
}
```
- 만약 이것이 허용된다면, 타입 소거 후에는 `new Comparable[2]`가 생성된다.
- 이 배열을 `String[]`변수에 대입하려고 하면, 런타임에 `ClassCastException`이 발생한다. `Comparable` 배열은 `String`배열의 하위 타입이 아니기 때문이다.

배열을 클래스 내부의 `private` 필드로만 사용하고 외부로 직접 노출하지 않는다면, `Object[]`를 사용하고 꺼낼 때 캐스팅하는 방식으로 우회할 수 있다.
```java
public class ArrayList<E> {
	private Object[] elements; 
	
	@SuppressWarnings("unchecked")
	public E get(int n){
		return (E) elements[n];
	}
}
```
---
복습용으로 적어보자면
위의 예시 코드에서 메서드 내부에서는 실질적인 검사나 변환이 일어나지 않는다. 
이때는 컴파일러가 호출하는 쪽에 캐스팅 코드를 자동으로 심는다.
브리지 메서드는 상속/오버라이딩 관계에서 다형성을 유지할 때 주로 쓰인다.

또한, Unchecked cast 경고가 뜨는 이유는, 개발자는 E 를 반환한다고 하지만, 컴파일러 입장에서는 런타임에 E가 뭔지 몰라서 검사를 못하기 때문이다. 

---

배열을 반환(`T[]`)해야 할 때 올바른 해결책은 사용자가 타입 정보를 제공해주어야 한다는 것이다. 컴파일러나 JVM은 `T`가 무엇인지 모르기 때문이다. 
: 1. 배열 생성자 참조 (권장되는 방법)
```java
public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr, T... a)
{
	T[] result = constr.apply(2);
	// ...
	return result;
}
...
//사용
String[] names = ArrayAlg.minmax(String[]::new, "Tom","Dick","Harry");
```

: 2. reflection, 전달받은 파라미터 배열(`a`)의 클래스 정보를 이용하여 새 배열을 만든다.
```java
public static <T extends Comparable> T[] minmax(T... a)
{
	var result = (T[]) Array.newInstance(a.getClass().getComponentType(),2);
	// ...
}
```


`ArrayList`의 `toArray()` 메서드도 같은 문제에 직면한다. 그래서 두 가지 버전이 있다.
- 1. `Object[] toArray()` : 제네릭 타입을 모르므로 `Object` 배열 반환
```java
ArrayList<String> list = new ArrayList<>();
list.add("A");

Object[] objects = list.toArray();

String[] strings = (String[]) list.toArray(); // 에러
```
- 2. `T[] toArray(T[] a)` : 사용자가 넘겨준 배열 `a`의 타입을 보고, 그 타입에 맞는 배열을 반환. 
```java
ArrayList<String> list = new ArrayList<>();
list.add("A");

String[] strings = list.toArray(new String[0]);

System.out.println(Strings[0]);
```
- 사용자가 빈 배열 혹은 크기가 맞는 배열을 인자로 넘겨준다. 이 배열은 데이터를 담을 그릇 역할도 하지만, 가장 중요한 역할은 타입 정보를 메서드 안으로 전달하는 것이다. 
	- 메서드 내부에서 사용자가 넘겨준 배열`a`의 타입을 확인한다(`a.getClass()`)
	- `ArrayList`의 크기보다 `a`가 작으면? -> `a`와 똑같은 타입의 새 배열을 내부에서 생성
	- `ArrayList`의 크기보다 `a`가 크거나 같으면? -> `a`를 그대로 재사용해서 데이터를 넣는다

## Type Variables Are Not Valid in Static Contexts of Generic Classes

제네릭 클래스의 static 필드나 static 메서드에서는 타입변수를 사용할 수 없다. 

```java
public class Singleton<T>
{
	private static T singleInstance; //에러
	
	public static T getSingleInstance() //에러
	{
		if(singleInstance == null) ...
		return singleInstance;
	}
}
```

이유틑 타입 소거가 일어나면 `T`의 실제 타입에 상관없이 JVM 상에서는 오직 하나의 `Singleton` 클래스만 존재하기 때문이다. 
- `static` 필드는 클래스당 하나만 존재하고 모든 인스턴스가 공유한다.
- 만약 위 코드가 허용된다면, `singleInstance` 필드는 어떤 타입일지 명확하지가 않으므로 공유될 수가 없다.


## You Cannot Throw or Catch Instances of a Generic Class

제네릭 예외 클래스 정의 불가
: 제네릭 클래스가 `Throwable`을 상속받으려 하면 컴파일 에러가 발생한다.
```java
public class Problem<T> extends Exception {...} //에러 
```

catch 블록에서 타입 변수 사용 불가
: `catch` 절에서 타입 변수 `T`를 잡을 수 없다
```java
public static <T extends Throwable> void doWork(Class<T> t){
	try{
		// ...
	}
	catch(T e){ // 에러
		// ...
	}
}
```
- 런타임에는 타입 소거로 인해 `T`가 무엇인지 알 수 없다. 예외 처리는 런타임에 발생한 예외의 타입을 확인해야 하는데, 타입 정보가 없으니 불가능하다.

throws 절에서는 사용 가능
: 단, 메서드 선언부의 `throws` 절에서는 타입 변수를 사용할 수 있다.
```java
public static <T extends Throwable> void doWork(T t) throws T
{
	try{
		//...
	}
	catch (Throwable realCause){
		t.initCause(realCause);
		throw t;
	}
}
```


## You Can Defeat Checked Exception Checking

자바는 Checked Exception을 반드시 처리해야 한다. 하지만 제네릭과 타입 소거의 허점을 이용하면 이 규칙을 우회할 수 있다. 이를 흔히 **Sneaky Throw** 라고 한다.

다음 메서드를 살펴보자
```java
@SuppressWarnings("unchecked")
static <T extends Throwable> void throwAs(Throwable t) throws T {
	throw (T) t;
}
```
이 메서드가 Task 라는 인터페이스에 포함되어 있다고 한다면 
`Task.<RuntimeException>throwAs(e)` 와 같이 호출할 수 있다. 
- 컴파일러는 `T`를 `RuntimeException`으로 간주한다.  `RuntimeException`은 Unchecked Exception 이므로 예외 처리를 강제하지 않는다.
- 런타임에서 `(T)`는 타입 소거로 인해 캐스팅의 의미가 없어진다. 결국 원래 예외 `e`가 그대로 던져진다.

활용 예제: Runnable에서 체크 예외 던지기
`Runnable.run()` 메서드는 체크 예외를 던질 수 없도록 설계되어 있다. 보통은 내부에서 `try-catch`로 감싸서 처리해야 하지만, 위 기법을 사용하면 코드를 간결하게 만들 수 있다.
: Runnable 인터페이스
```java
@FunctionalInterface
public interface Runnable {
    // 매개변수도 없고, 반환값(return)도 없는 메서드
    public abstract void run();
}
// 메서드 선언부에 throws Exception이 없음
// 따라서 IOException이나 SQLException 같은 체크 예외가 발생하면, 반드시 내부에서 try-catch로 해결해야 한다. 밖으로 던질 수 없음.
```
: 예시 코드
```java
interface Task {
    void run() throws Exception;

    // 체크 예외를 언체크 예외처럼 던지는 헬퍼 메서드
    @SuppressWarnings("unchecked")
    static <T extends Throwable> void throwAs(Throwable t) throws T {
        throw (T) t;
    }

    // Task를 Runnable로 변환해주는 어댑터
    static Runnable asRunnable(Task task) {
        return () -> {
            try {
                task.run();
            } catch (Exception e) {
                // 여기서 컴파일러를 속여 체크 예외를 그냥 던져버림
                Task.<RuntimeException>throwAs(e);
            }
        };
    }
}

public class Test {
    public static void main(String[] args) {
        var thread = new Thread(Task.asRunnable(() -> {
            Thread.sleep(1000); // InterruptedException(체크 예외) 발생 가능
            System.out.println("Hello, World!");
            throw new Exception("Check this out!");
        }));
        thread.start(); 
    }
}
```


## Beware of Clashes after Erasure

타입 소거 후에 메서드 시그니처가 동일해지는 경우, 충돌이 발생하여 컴파일 에러가 난다/

`Pair<T>` 클래스에 `equals(T value)` 를 만든다고 가정해보자
```java
public class Pair<T>{
	public boolean equals(T value){...}
}
```
- 타입 소거 후, `T` 는 `Object` 가 된다.
- 이는모든 클래스가 상속받는 `Object.equals(Object)`와 시그니처가 정확히 겹친다. 오버라이딩을 의도한 게 아니라면 메서드 이름을 바꿔야 한다.

서로 다른 타입 파라미터를 가진 동일한 인터페이스를 동시에 상속 받을 수 없다.
```java
class Employee implements Comparable<Employee> {...}

class Manager extends Employee implements Comparable<Manager> { ...} //에러
```
- `Manager` 는 `Comparable<Employee>`와 `Comparable<Manager>` 모두를 가지게 된다.
- 이는 브리지 메서드 때문에 문제가 된다.
- 1. `Employee` 는 `compareTo(Object)` -> `compareTo(Employee)` 로 연결되는 브리지 메서드를 생성한다
- 2. `Manager` 는 `compareTo(Object)` -> `compareTo(Manager)` 로 연결되는 브리지 메서드를 생성한다
- 3. `Manager` 클래스 안에 `public int compareTo(Object other)` 라는 똑같은 시그니처의 메서드가 두 개 생기게 되어 컴파일 에러가 발생한다.






















































