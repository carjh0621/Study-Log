제네릭 시스템이 너무 엄격하면 사용하기가 불편하다. 자바 설계자들은 이 문제를 해결하면서도 타입 안정성을 해치지 않는 방법을 고안해냈다. 이것이 바로 와일드카드 타입이다.


## The Wildcard Concept

엄격한 제네릭의 문제점을 보자. 예를 들어 직원 쌍을 받아서 이름을 출력하는 `printBuddies`라는 메서드를 만든다고 한다면
```java
public static void printBuddies(Pair<Employee> p){
	Employee first = p.getFirst();
	Employee second = p.getSecond();
	System.out.println(first.getName() + "and" + second.getName() + " are buddies.");
}
```
- `Pair<Manager>` 객체를 이 메서드에 넘길 수 없다는 단점이 있다.
- 이유는 `Pair<Manager>` 는 `Pair<Employee>`의 하위 타입이 아니기 때문이다.

해결책은 와일드카드 타입을 사용하는 것이다.
```java
public static void printBuddies(Pair<? extends Employee> p)
```
- `Pair<? extends Employee>` 는 `Employee`본인 또는 그 하위 클래스를 타입 파라미터로 갖는 모든 `Pair`를 의미한다. 따라서 이 메서드는 이제 `Pair<Employee>` 뿐만이 아니라 `Pair<Manager>`도 받을 수 있다.


와일드카드를 사용하면 유연함을 얻는 대신, 데이터를 변경하는데 있어서 제약이 생긴다. 
예를 들어 `Manager`인 ceo와 cfo를 담은 `Pair<Manager>`를 만들고 이를 와일드카드 참조변수에 대입한다고 가정해 보자
```java
var managerBuddies = new Pair<>(ceo,cfo);
Pair<? extends Employee> wildcardBuddies = managerBuddies;
```
이제 와일드카드 변수를 통해 데이터를 조작해보자
``` java
wildcardBuddies.setFirst(lowlyEmployee); // 컴파일 에러
```
- 컴파일러 입장에서는 `wildcardBuddies`는 `? extends Employee`를 담고 있다. 즉, 구체적인 타입이 뭔지 모른다.
- 만약 `setFirst`에 `Employee`를 넣는 것을 허용한다면, 실제 객체가 `Pair<Manager>`인 경우 일반 직원이 매니저 리스트에 들어가는 Corruption(오염)이 발생한다. 심지어 `Manager`객체를 넣으려 해도 컴파일러는 변수의 구체적인 타입을 모르기 때문에 허용되지 않는다.
- 따라서 컴파일러는 `null`을 제외한 모든 객체의 입력을 허용하지 않는다.
```java
Employee first = wildcardBuddies.getFirst(); 
```
- 와일드카드 변수에 무엇이 들어있든, 그것들이 모두 `Employee`의 자식이므로 `Employee`변수에 담는 것은 안전하다. 


## Supertype Bounds

`? super Manager`
: `Pair<? super Manager>`는 `Manager` 및 `Manager` 의 조상 클래스를 타입 파라미터로 갖는 `Pair`를 의미한다.

supertype bound는 `? extends`와 달리 쓰기는 되지만 읽기는 까다롭다

컴파일러가 이 변수가 정확이 어떤 타입인지는 모르지만 최소한 `Manager`나 그 하위타입을 담을 수 있는 변수임을 확신할 수 있기 때문이다
```java
void setFirst(? super Manger) // 내부적으로 컴파일러가 인식하는 setter의 모습
//...
result.setFirst(new Manager());
result.setFirst(new Executive()); // Manager의 자식
```

반면, 이 변수에서 읽을 때는 변수의 타입이 어떤 타입인지 모르기 때문에 (`Pair<Object>`일수도 있음) 오직 `Object`타입으로만 꺼낼 수 있다.

예를들어 관리자 배열에서 보너스를 가장 적게 받는 관리자와 가장 많이 받는 관리자를 찾아서 하나의 `Pair`에 담는 메서드 (`minmaxBonus`)를 만든다고 하자.
```java
public static void minmaxBonus(Manager[] a, Pair<? super Manager> result)
{
	if(a.length ==0) return;
	
	Manager min = a[0];
	Manager max = a[0];
	
	for(int i=1; i<a.length; i++){
		if(min.getBonus() > a[i].getBonus()) min = a[i];
		if(max.getBonus() < a[i].getBonus()) max = a[i];
	}
	
	result.setFirst(min);
	result.setSecond(max);
}
```


`T extends Comparable<? super T>`
어떤 타입 `T`의 배열에서 최솟값과 최댓값을 찾는 제네릭 메서드 `minmax`를 만든다고 해보자. `T`는 서로 비교 가능해야하므로 `Comparable`을 구현해야 한다.
- `<T extends Comparable<T>>` 의 경우 `String`클래스는 잘 작동하지만 `LocalDate`클래스는 이 조건을 통과하지 못한다.
	- `LocalDate`는 `ChronoLocalDate`라는 상위 인터페이스를 구현한다. 그리고 `Comparable`은 `LocalDate`가 아니라 그 부모인 `ChronoLocalDate`에 구현되어 있다.
	- 즉 `LocalDate`는 `Comparable<LocalDate>`가 아니라 `Comparable<ChronoLocalDate>` 이다.
- 따라서 제네릭 선언을 다음과 같이 해야한다.
```java
public static <T extends Comparable<? super T>> Pair<T> minmax(T[] a)
// T는 T 본인 또는 T의 조상 타입과 비교 가능한 타입이어야 한다.
```

이 개념은 람다와 함수형 인터페이스에서도 자주 쓰인다. 
예를들어, `ArrayList<Employee>` 에서 직원의 해시코드가 홀수인 사람을 리스트에서 지우려고 해보자.
```java
ArrayList<Employee> staff = ...;

Predicate<Object> oddHashCode = obj -> obj.hashCode() % 2 != 0;

staff.removeIf(oddHashCode);
```
- `removeIf`가 만약 `Predicate<Employee>` 만 받도록 설계되었다면, 예시처럼 `Object`를 다루는 필터를 재사용할 수 없었을 것이다.
- 하지만 자바 설계자들은 이를 `default boolean removeIf(Predicate<? super E> filter)` 로 설계했기 때문에, `Employee`의 상위타입인 `Object`를 처리하는 필터도 유연하게 넘길 수 있다.
---
`Predicate` : 함수형 인터페이스, 무언가를 입력받아서 T/F를 반환하는 판독기 역할
`Predicate<Object> oddHashCode = obj -> obj.hashCode() % 2 != 0;`
: obj 를 입력받으면 해시코드를 검사해 홀짝을 판별하는 객체를 임시로 생성해 oddHashCode 변수가 참조
`removeIf` : 컬렉션에 내장된 청소기 같은 메서드? predicate를 장착해주면 됨

---



## Unbounded Wildcards

앞서서 제한이 있는 와일드 카드들을 보았다. 이번에는 아무런 제한이 없는 와일드카드인 `<?>`에 대해서 볼것이다.

`Pair<?>`, `Pair`
- 원시 타입 `Pair` : 타입 검사를 포기한 상태이다. 위험성이 있다.
- Unbounded Wildcards `Pair<?>` : 컴파일러는 이를 어떤 특정 타입이긴 한데 그게 뭔지 모른다고 인식한다. 따라서 데이터 오염을 막기 위해 null을 제외하고 데이터 수정을 막는다.

값을 넣지도 못하고, 꺼내도 `Object` 밖에 안되는 이 타입을 쓰는 이유?
: 실제 타입이 무엇이든 전혀 상관없는, 아주 단순한 작업을 할 때 유용하다.
: 예를 들어 `Pair` 객체 안에 `null`이 하나라도 있는지 검사하는 메서드를 작성한다고 생각해보자
- 이 메서드는 안의 데이터가 `String`인지 `Employee`인지 알 필요가 없다. 그저 꺼내서 null인지 확인만 하면 되므로 `<?>`가 적합하다. 


## Wildcard Capture

예를 들어 `Pair`안에 있는 두 값의 자리를 바꾸는 메서드를 만든다고 해보자. 
```java
public static void swap(Pair<?> p){
	? t = p.getFirst(); // 컴파일 에러
	
	p.setFirst(p.getSecond());
	p.setSecond(t);
}
```
문제는 `?`는 실제 존재하는 타입 이름이 아니라 일종의 기호이다. 즉 `? t` 처럼 변수를 선언할 때 타입으로 사용할 수 없다.

이 문제를 해결하려면 `?`를 구체적인 타입 변수 `T`로 묶어줄(Capture) 필요가 있다. 
이를 위해 제네릭 헬퍼 메서드를 새로 만든다. 
```java
public static <T> void swapHelper(Pair<T> p){
	T t = p.getFirst();
	p.setFirst(p.getSecond());
	p.setSecond(t);
}

public static void swap(Pair<?> p){
	swap.Helper(p); 
}
```
- `swap` 메서드에 `Pair<String>`이 전달되었다고 치자. 컴파일러는 `swapHelper`를 호출할 때 `p`의 실제 타입이 `String`임을 파악하고 `T`를 `String`으로 대체한다. 결과적으로 헬퍼 메서드 안에서는 구체적인 타입을 사용하여 임시변수(`T t`)를 만들어 안전하게 값을 교환할 수 있다.
























































