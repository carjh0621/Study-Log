

## Separating Collection Interfaces and Implementation

현대적인 자료구조 라이브러리의 중요한 원칙은 interface와 implementation을 분리하는 것이다.
교재에서는 이를 큐를 통해 설명한다.
큐의 인터페이스는 FIFO, 데이터를 뒤에 추가하고 앞에서 추출하는 행위를 의미한다.
```java
public interface Queue<E>{
	void add(E element); //tail에 요소를 추가
	E remove(); // head에서 요소를 제거
	int size();
}
```
Implementaion
- 1. Circular Array Queue
```java
public class CircularArrayQueue<E> implements Queue<E> {
	private int head;
	private int tail;
	private E[] elements;
	
	CircularArrayQueue(int capacity) {...}
	public void add(E element){...}
	public E remove() {...}
	public int size() {...}
}
```
- 2. Linked List Queue
```java
public class LinkedListQueue<E> implements Queue<E> {
   private Link head; // 첫 번째 노드를 가리키는 포인터
   private Link tail; // 마지막 노드를 가리키는 포인터

   LinkedListQueue() { ... }
   public void add(E element) { ... }
   public E remove() { ... }
   public int size() { ... }
}
```

이를 실제로 사용할 때 
```java
Queue<Customer> expressLane = new CircularArrayQueue<>(100);
```
위와 같이 사용 가능하다. 핵심은 필요에 따라 생성자 부분만 `new LinkedListQueue<>()`로 교체하면 된다는 것이다 .이것이 인터페이스와 구현을 분리하는 이유이다.

API를 보다보면 `AbstractQueue`같은 추상 클래스가존재한다. 이는 개발자가 특수한 큐를 처음부터 다 구현하는 수고를 덜기 위해 기본적이고 반복적인 로직을 미리 구현해 둔 클래스이다.


## The `Collection` Interface

자바에서 데이터를 담는 역할을 하는 대부분의 클래스는 `Collection` 인터페이스를 조상으로 가진다.

`Collection`인터페이스는 많은 메서드를 가지고 있지만, 교재는 두 가지 메서드를 다룬다.
```java
public interface Collection<E> extends Iterable<E> {
	boolean add(E element); // 컬렉션에 요소 추가
	Iterator<E> iterator(); // 컬렉션 내부의 요소들을 순차적으로 방문할 수 있게 해주는 반복자
	//...
}
```


## Iterators

배열은 index를 통해 접근하면 되지만, `LinkedList`, `HashSet`, `TreeSet` 등의 내부 구조가 복잡한 자료구조들을 순서대로 데이터를 꺼내기가 애매하다. 사용자가 내부 구조를 알 필요 없이, 다음 요소를 요청할 수 있도록 하는 표준 규격이 `Iterator`이다.

Cursor 모델
: Iterator은 요소를 가리키는 것이 아니라 요소와 요소 사이(Between Elements)를 가리키는 것이다. 
```java
public interface Iterator<E>{
	E next();
	boolean hasNext();
	void remove();
	default void forEachRemaining(Consumer<? super E> action);
}
```
- next : 반복자를 다음 위치로 이동, 넘어간 요소를 반환한다. 더 이상 요소가 없는데 호출하면 `NoSuchElementException`을 발생시킨다. 
- remove: next 메서드가 가장 마지막으로 반환했던 요소를 삭제한다. 
	- next를 호출하지 않고 remove를 호출하면 `IllegalStateException`이 발생한다.
	- 한 번의 next에 대해 remove는 딱 한번만 호출할 수 있다.


`for each` 루프와 `Iterable`
: 향상된 for문은 컴파일러가 `Iterator`로 바꿔치기한다.
```java
for (String element : c){
	//...
}

-> 컴파일러가 변환

Iterator<String> iter = c.iterator();
while(iter.hasNext()){
	String element = iter.next();
	// ...
}
```


## Generic Utility Methods

컬렉션의 일상적인 기능 `isEmpty`, `clear`, `contains` 등은 반드시 필요하다
이를 `Collection`인터페이스에 전부 정의해두면, 사용자는 편해지지만 새로운 컬렉션 클래스를 만들려는 라이브러리 개발자는 오버라이드를 많이 해야하는 불편함이 있다. 

이를 해결하기 위해 등장한 개념이 제네릭 알고리즘과 추상 클래스이다.

컬렉션의 내부 구조가 배열이든 해시 테이블이든, `Iterator`만 있으면 모든 요소를 순회할 수 있다.
즉 특정 요소가 존재하는지 확인하는 로직은 내부구조를 몰라도 다음과 같이 범용적으로 작성할 수 있다.
```java
public static <E> boolean contains(Collection<E> c, Object obj){
	for(E element: c){
		if(element.equals(obj)) return true;
	}
	return false;
}
```

위와 같은 범용 로직을 미리 구현해둔 `AbstractCollection`이라는 추상 클래스를 제공한다.
```java
public abstract class AbstractCollection<E> implements Collection<E> {
   //추상 메서드 -> 자식 클래스에 구현 강제
   public abstract Iterator<E> iterator();
   public abstract int size();

   // 일상적인 기능은 미리 구현
   public boolean contains(Object obj) {
      for (E element : this) // 자식 클래스가 구현할 iterator()가 호출됨
         if (element.equals(obj)) return true;
      return false;
   }
}
```
- 새로운 큐나 리스트를 만들고 싶다면, `Collection` 인터페이스를 처음부터 구현할 필요 없이 `AbstractCollection`을 상속받으면 된다. `iterator()`, `size()` 만 구현하면 됨 

자바 7 까지는 인터페이스 안에 body를 넣을 수 없었다. 그래서 `AbstractCollection`같은 별도의 추상 클래스를 만들어야 했다.
자바 8 부터는 인터페이스 안에 `default`키워드를 붙이면 구현 코드를 직접 넣을 수 있게 되었다.
`words.removeIf(w -> w.length()<3);` : 원래 `removeIf(Predicate filter)` 인데, 인터페이스 자체가 기본 로직을 가질 수 있게 되어 람다식으로 처리 가능하다. 

---
우선 향샹된 for 문의 : 대상은 배열과 `Iterable` 인터페이스를 구현한 객체 두가지로 제한된다.

AbstractCollection은 추상 클래스이므로 스스로 `new`를 통해 생성될 수 없다.
`for (E element : this)` 여기에서 this는
이를 상속받은 자식 클래스의 인스턴스를 의미한다.

---























