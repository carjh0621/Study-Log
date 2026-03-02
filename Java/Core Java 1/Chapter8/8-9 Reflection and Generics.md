Reflection은 실행 중에 프로그램의 클래스, 메서드, 필드 등을 분석하고 조작할 수 있는 기능이다. 
하지만 제네릭과 리플렉션이 만나면 문제가 생긴다.
제네릭은 타입소거가 되기 때문에 리플렉션으로 알아낼 수 있는 정보는 제한적이다. 
그럼에도 불구하고 자바 설계자들은 리플렉션 API 자체에 제네릭을 적용하여, 좀 더 안전하고 편리하게 코딩할 수 있도록 만들었다.

## The Generic `class` Class

자바에는 특정 클래스의 메타정보를 담고 있는 `Class`라는 클래스가 있다. 자바 5부터 `Class` 클래스 자체가 제네릭 클래스로 변경되었다. (`Class<특정 클래스>`)

`Class`를 제네릭으로 바꾼 가장 큰 이유는 불필요한 형변환을 없애기 위해서이다. `Class<T>` 객체의 메서드들이 이제 `Object`가 아닌 구체적인 타입 `T`를 반환할 수 있게 되었다. 

제네릭이 적용된 대표적인 리플렉션 API 메서드들

| 메서드 (반환 타입 포함)                       | 설명                                                          |
| ------------------------------------ | ----------------------------------------------------------- |
| `T newInstance()`                    | 매개변수가 없는 기본 생성자를 호출하여 `T` 타입의 새 인스턴스를 반환. (예전엔 `Object` 반환) |
| `T cast(Object obj)`                 | 주어진 객체를 `T` 타입으로 변환하여 반환. 불가능하면 예외를 던진다.                    |
| `T[] getEnumConstants()`             | 이 클래스가 열거형(Enum)인 경우, 그 상수들을 `T` 타입의 배열로 반환.                |
| `Class<? super T> getSuperclass()`   | 이 클래스의 부모 클래스 정보를 반환한다. (부모이므로 `? super T` 사용)              |
| `Constructor<T> getConstructor(...)` | 특정 매개변수를 갖는 `public` 생성자를 반환한다. (생성자 클래스도 제네릭이 되었다)         |

---
교재에서는 `cast` 메서드가 실패할 경우 `BadCastException`을 던진다고 되어 있으나, 자바 표준 API 에서는 `ClassCastException`을 던진다.

`T newInstance()`는 자바9부터 사용 중지되었다. 현재는 `Class.getDeclaredConstructor().newInstance()`를 사용하는 것이 옳다. 

---


## Using `class<T>` Parameters for Type Matching

이전의 내용에 의하면 제네릭 메서드 안에서는 `new T()`를 호출할 수 없었다. 이유는 런타임에 `T`가 소거되어 무엇인지 알 수 없기 때문이었다.

이 문제를 우회하기 위해, 호출하는 쪽에서 런타임 타입 정보인 `Class<T>` 객체를 메서드의 파라미터로 직접 넘겨주는 패턴을 주로 사용한다. 
```java 
//호출자
Pair<Employee> p = makePair(Employee.class);
// Employee.class 즉 Class<Employee>를 명시적으로 넘긴다
```
호출받는 메서드는 이 정보를 이용해 안전하게 객체를 생성한다.
```java
public static <T> Pair<T> makePair(Class<T> c) 
	throws InstantiationException, IllegalAccessException
{
	// c.newInstance()sms Object가 아니라 T를 반환한다.
	// 따라서 형변환을 할 필요가 없다.
	return new Pair<>(c.newInstance(), c.newInstance());
}
```



## Generic Type Information in the Virtual Machine

이전 내용들에서 자바의 제네릭은 타입 소거를 통해 런타임에 모든 제네릭 정보를 지운다고 배웠다.

실제로는 객체 자체는 자신이 생성될 때의 제네릭 타입을 기억하지 못하지만, 클래스 설계도와 메서드 선언부에는 제네릭 타입의 Signature가 바이트코드 수준에서 저장되어 있다.

예를 들어, 다음과 같은 제네릭 메서드가 있다고 해보자
```java
public static <T extends Comparable<? super T>> T min(T[] a)
```
타입 소거 후에 이 메서드는 `Public static Comparable min(Comparable[] a)` 로 바뀐다. 
하지만 리플렉션 API를 사용하면, 지워지기 전의 원래 선언 모습에 대한 모든 정보를 파악할 수 있다.
- 제네릭 파라미터 `T`가 존재한다는 것
- `T`가 `Comparable`을 상속받는다는 것
- `Comparable`안에 와일드카드 `?`가 있다는 것
- `?`가 `super T`라는 제한을 갖는다는 것
- 파라미터가 `T[]` 라는 것
단, 리플렉션으로 실제로 지금 실행 중인 객체나 메서드 호출이 어떤 구체적인 타입으로 해석되었는지는 알 수없다. 오직 선언 당시의 모습만 복원할 수 있다.

리플렉션을 통해 제네릭 정보를 읽어오기 위해, 자바는 `java.lang.reflect` 패키지에 `Type`이라는 인터페이스를 제공한다. 제네릭이 관련된 모든 요소는 이 `Type`의 하위 인터페이스들로 표현된다. 
- `Class` 클래스 : 제네릭이 없는 일반적인 클래스 (`String` 등)
- `TypeVariable` 인터페이스 : 제네릭 타입 변수 자체 (`T`) 
- `WildcardType` 인터페이스 : `? super T` 같은 와일드카드를 나타낸다.
- `ParameterizedType` 인터페이스 : 타입 파라미터가 채워진 제네릭 클래스나 인터페이스 (`Comparable<? super T>` 등)
- `GenericArrayType` 인터페이스 : 제네릭 요소로 이루어진 배열을 의미 (`T[]`)
이 인터페이스들을 재귀적으로 조합하여 분석하면 복잡한 제네릭 선언도 복원할 수 있다.



## Type Literals

예를 들어 객체의 타입에 따라 다른 동작을 수행하게 만들려고 할 때. 
(`ArrayList<Integer>` 와 `ArrayList<Character>` 가 화면에 출력되는 방식에 차이를 주고 싶다고 하자)
런타임에는 타입 소거 때문에 둘 모두 `ArrayList`로 보인다. `Class` 객체를 사용하려 해도 둘 다 `ArrayList.class`일 뿐이다.

이 문제를 해결하기 위해 고안된 기법이 타입 리터럴이다. 객체를 생성할 때 뒤에 중괄호를 붙이는 것이다. `var type = new TypeLiteral<ArrayList<Integer>>(){};`
- 중괄호를 붙이면, `TypeLiteral` 객체를 생성하는 것이 아니라 `TypeLiteral<ArrayList<Integer>>` 를 상속받은 익명 서브클래스를 새로 생성하는 것이다
- 타입 소거 시에, 객체의 제네릭은 지워져도 클래스 상속관계의 제네릭 정보는 바이트 코드에 보존된다.
- 따라서, 이 익명 서브클래스의 부모 클래스 정보를 리플렉션으로 탐색하면 `ArrayList<Integer>` 라는 구체적인 제네릭 타입을 런타임으로도 알아낼 수 있다.



































