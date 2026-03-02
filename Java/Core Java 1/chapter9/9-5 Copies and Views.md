보통 데이터를 저장하기 위해 `ArrayList` , `HashSet`같은 객체를 만든다. 데이터를 기존 저장소에서 새로운 저장소로 옮기는 등의 복사비용이 때로는 매우 클 수도 있다. 이는 메모리 낭비와 성능 저하를 일으킬 수 있다.
이 문제를 해결하기 위해 등장한 것이 View이다.
- 뷰는 데이터를 보여주기만 할 뿐이다. 뷰 객체의 메서드를 호출하면, 뷰는 자신이 데이터를 처리하는 대신 원본 컬렉션에 그 작업을 그대로 전달한다.
앞서 나온 `Map`의`keySet()` 메서드가 그 예이다. 이 메서드는 `Map`에 저장된 모든 키를 `Set` 형태로 반환한다. 이때 새로운 `Set`을 만들어서 키를 복사하는 것이 아니다. 대신 원본 `Map`의 키들만 `Set`의 규격에 맞춰서 보여주는 뷰 객체를 반환한다.

## Small Collections

알고리즘 문제를 풀거나 할 때, 데이터가 몇개 없는 `List`, `Set` 을 만들기 위해 매번 new -> add 를 반복하는 것은 번거로운 일이다. 자바 9부터는 간단한 정적 팩토리 메서드가 도입되었다.

`List`, `Set`, `Map` 인터페이스에 정의된 `of()` 메서드를 사용하면 값을 나열하는 것만으로 컬렉션이 완성된다.
- `List<String> names = List.of("Peter", "Paul", "Mary");`
- `Set<Integer> numbers = Set.of(2, 3, 5);`
- `Map<String, Integer> scores = Map.of("Peter", 2, "Paul", 3, "Mary", 5);` (키, 값)

이렇게 만들어진 컬렉션들은 매우 가볍고 빠르지만, 내부적으로 수정 불가능하게 설계된다.
- `add()`, `remove()`, `set()` 등을 호출하면 즉시 `UnsupportedOperationException` 에러가 발생한다. (읽기 전용)
- 요소나 키/값으로 `null`을 넣으면 생성 시점에 바로 에러가 난다.
- `Set.of(13, 13)`처럼 중복된 값을 넣거나, `Map`에 중복된 키를 넣으면 에러가 난다.
이 컬렉션에 데이터를 추가하거나 수정해야 한다면, 일반 컬렉션의 생성자에 감싸서 새로운 수정 가능한 객체로 복사해야한다.
```java
// 수정 불가능한 List를 통째로 ArrayList 생성자에 넣어 수정 가능한 리스트로 바꿈
var mutableNames = new ArrayList<>(List.of("Peter", "Paul", "Mary"));
```

부연
: 원래 `Set`, `Map`은 순서를 보장하지 않는 자료구조이다. 하지만 과거의 많은 프로그래머들은 특정 순서대로(우연히) 나오는 것에 의존해서 코드를 짜는 실수를 저질렀다고 함...
이를 예방하기 위해 자바9부터 `Set.of()`, `Map.of()`의 내부 순서를 JVM이 켜질 때마다 고의로 뒤섞도록 했다고 한다.

`List.of()`와 `Set.of()`는 내부적으로 매개변수가 0개부터 10개인 메서드가 각각 미리 만들어져 있고, 그 이상은 가변인자를 사용한다.
하지만 `Map.of()`는 가변인자를 쓸 수 없다. 자바 문법상 가변 인자는 하나의 타입만 계속 받아야 하는데, Map의 경우 키, 값이 반복적으로 들어오기 때문이다. 
11쌍 이상의 데이터를 한번에 초기화하려면 Map.Entry(키-값 객체)를 가변인자로 받는 `Map.ofEntri2es()`를 써야한다.
```java
import static java.util.Map.*;

Map<String, Integer> scores = Map.ofEntries(
	entry("Peter", 2),
	//... 10개 이상 추가 가능
);
```

리스트를 만들때 `"DEFAULT"`라는 문자열이 100개 들어있는 리스트가 필요하다면? 반복문을 100번 돌리지 말고 
```java 
List<String> settings = Collections.nCopies(100,"DEFAULT");
```
이 메서드는 문자열 객체를 메모리에 하나만 만들고, 뷰 리스트를 반환한다. 이 역시 수정은 불가능하다.


## Unmodifiable Copies and Views

새롭게 리스트나 맵을 다른 메서드나 클래스에 넘겨야 할 때가 있다.
이때, 넘겨받은 쪽에서 실수든 악의적이든 원본 데이터를 훼손하는 것을 막고싶을 수 있다. 이를 달성하는 방법은 크게 두가지로 나뉜다. 
1. 아예 새로운 복사본을 만들어 주는 것 (Unmodifiable Copies)
2. 뷰를 전달하는 것.

Unmodifiable Copies
: Java 10부터 도입된 `List.copyOf()`, `Set.copyOf()`, `Map.copyOf()` 메서드를 사용한다. 이 방식은 호출되는 순간의 원본 데이터를 바탕으로 새로운 독립적인 컬렉션을 메모리에 만든다. (스냅샷이라 봐도 무방하다)
이 복사본에 데이터를 추가/삭제하려 하면 `UnsupportedOperationException` 에러가 발생한다.
: 이 방식은 무조건 메모리를 낭비하지 않는다. 만약 원본 컬렉션이 이미 수정 불가능한 타입이라면 (`Set.of()` 등), 자바는 굳이 새 객체를 만들지 않고 원본의 주소를 그대로 반환한다. 

Unmodifiable Views
: `Collections` 유틸리티 클래스에서 제공하는 `Collections.unmodifiableList()`, `Collections.unmodifiableSet()` 등의 메서드를 사용한다. view 객체를 반환한다.
```java
var staff = new LinkedList<String>();
// staff에 데이터 추가...

// 다른 메서드에 넘길 때 유리 덮개를 씌워서 넘김
lookAt(Collections.unmodifiableList(staff));
```


주의사항
: 자바에서 객체의 내용이 같은지 비교할 때는 `equals()`를 쓴다. `Collections.unmodifiableList()`나 `unmodifiableSet()`으로 만든 뷰는, 원본 리스트나 집합의 `equals()` 동작을 정상적으로 물려받는다. (즉, 안의 데이터가 같으면 같다고 판별한다.) 
하지만, `Collections.unmodifiableCollection()`으로 만든 가장 상위 단계의 뷰는 내부 데이터 비교를 포기한다.
그 이유는 `Collection`이라는 가장 큰 범주에는 list와 set이 모두 들어올 수 있다는 점에서 기인한다.
만약 원본이 list인데, 뷰를 단순 `Collection`으로 씌운다면 이 뷰를 다른 Set과 비교했을 때 같다고 해야할지 다르다고 해야할지 애매하다.(순서가 달라도 데이터가 모두 일치할 수 있음)
그렇기에 자바 설계자들은 아예 내용물 비교를 막아버리고 단순히 메모리 주소가 같은 객체인가만 검사하도록 한다. (equals() 하면 실제 동작은  `==`와 동일해진다.)


## Subranges

뷰와 같은데 다른점은 원본 컬렉션의 특정 구간만 보여주는 view를 의미한다.

List의 경우 데이터간의 순서가 명확하다. 인덱스로 위치를 지정하면 되기 때문이다. 
문자열의 `substring`처럼 시작 인덱스는 포함하고, 끝 인덱스는 포함하지 않는다.
```java
List<String> staff = ...; // 직원 명단 (데이터가 100개 있다고 가정)
List<String> group2 = staff.subList(10, 20);
```
만약 `group2`의 데이터를 전부 삭제한다면 어떻게 될까? (`group2.clear()`)
-> `group2`가 비워지는 것 뿐만 아니라. 원본 `staff`리스트에서도 해당 번호의 데이터들이 삭제된다. 이처럼 `subList`는 특정 구간을 일괄 처리할 때 좋다.

`Set`, `Map`의 경우에는 인덱스라는 개념이 없다. 하지만 내부 데이터가 정렬되어 있는 구조.(`SortedSet`, `SortedMap`)의 경우에는 특정 구간을 자를 수 있다. 이때는 위치가 아니라 값의 크기를 이용해서 자른다. 
Set을 예시로 보면
- `subSet(from, to)`: `from` 값 이상부터 `to` 값 미만까지의 뷰를 반환한다. (기본적으로 시작점은 포함, 끝점은 미포함)
- `headSet(to)`: 처음부터 `to` 값 미만까지의 뷰를 반환한다.
- `tailSet(from)`: `from` 값 이상부터 끝까지의 뷰를 반환한다. 
Set을 Map으로 바꾸면 Map에도 적용된다.

위의 메서드들은 공통적으로 ~이상 ~미만 의 방식으로 고정되어있다. (주어진 인자에 따라 구절이 생략되긴 함). 
여기서 좀 더 유연하게 ~초과 혹은 ~이하의 요구사항도 포함하기 위해, 자바 6에서 `NavigableSet`과 `NavigableMap`이 나왔다. 여기에는 각 경계값을 포함할지 말지를 참,거짓을 통해 정할 수 있다.
```java
// from 값을 포함할지(fromInclusive), to 값을 포함할지(toInclusive) 직접 결정
NavigableSet<E> subSet(E from, boolean fromInclusive, E to, boolean toInclusive)

// 끝점 포함 여부 결정
NavigableSet<E> headSet(E to, boolean toInclusive)

// 시작점 포함 여부 결정
NavigableSet<E> tailSet(E from, boolean fromInclusive)
```

---
`subList`를 만들어 둔 상태에서 원본리스트의 크기를 직접 변경하면 안된다.(add, remove x)
예를들어 100개의 데이터를 가진 리스트의 10~20번 뷰를 만들어 놓았다 해보자
- 1. 만약 0번 인덱스에 새로운 데이터를 추가했다면, 뷰가 원래 보고 있던 데이터들이 한칸씩 밀렸는데도 여전히 10~20만 보고 있다. -> 뷰가 보여줘야할 원래 데이터의 정확한 위치를 잃어버린 상황
- 2. 만약 맨 뒤에 추가했다면, 논리적으로는 상관이 없으나, 자바는 허용하지 않는다. 앞서 Iterator 순회할 때 리스트를 변경한다면 `modCount`가 달라져서 예외가 발생한다고 했다. 비슷하게 뷰도 원본 리스트의 현재 `modCount`값을 자신의 메모리에 복사해놓는다. 뷰를 통한 접근을 할때마다 검사해서 값이 같지 않으면 예외가 발생한다. (Fail-Fast 전략)

---


## Checked Views

자바에서 `ArrayList<String>`을 만들면 문자열만 들어갈 수 있다. 하지만 제네릭이 없던 버전과의 호환성 때문에, 자바는 원시 타입으로의 변환을 허용한다.
이로 인해 Smuggling(밀반입)이  가능해진다. 
```java
ArrayList<String> strings = new ArrayList<>();
ArrayList rawList = strings; // 제네릭을 없앤 변수에 연결 

rawList.add(new Date()); // String 리스트에 Date 객체가 들어감
```
문제는 들어갈 때 에러가 안 나고, 나중에 이 리스트에서 데이터를 꺼내서 쓸 때 `ClassCastException`이 터진다는 것이다. 에러의 원인을 추적하기가 힘들어진다.

해결책
: `Collections.checkedList(strings, String.class)`로 뷰를 씌운다. 이 뷰는 데이터를 추가할 때마다 실시간으로 타입이 맞는지 검사한다.
만약 다른 타입이 들어오려하면, 나중이 아니라 데이터를 넣으려는 순간에 에러를 던진다.(Fail-Fast)
단, `Pair<String,Integer>`처럼 복잡하게 중첩된 제네릭 타입까지는 완벽하게 검사하지 못한다.




## Synchronized Views

웹 서버처럼 멀티스레드 환경에서, Thread-safe가 보장이 안된 자료구조에 데이터를 추가하려고 하면, 동시성 문제가 발생할 수 있다.

자바 설계자들은 뷰를 활용한 해결책을 제공한다.
```java
Map<String, Integer> map = Collections.synchronizedMap(new HashMap<>());
```
이렇게 뷰를 씌우면, 이 뷰는 한 번에 오직 하나의 스레드만 맵에 접근할 수 있도록 입구에 락을 건다.


## A Note on Optional Operations

앞서 수정 불가능한 뷰에서 `.add()`를 호출하면 에러가 난다고 배웠다. 그런데 인터페이스는 그 정의가 이 클래스는 이 메서드들을 반드시 정상적으로 실행할 수 있어야 한다는 약속이라 볼 수있다. 그런데 호출했는데 에러를 던지는건 옳은 설계와는 거리가 있다. 

이론적으로 완벽하려면 다음과 같이 인터페이스를 쪼개야했다.
- `ReadOnlyList` (get만 가능)
- `FixedSizeList` (get, set 가능 / add, remove 불가)
- `MutableList` (전부 가능)
하지만 이렇게 설계하면 Set, Map, Queue 등 모든 구조에 대해 인터페이스 개수가 3배로 증가하게 된다. 설계자들은 이 복잡성을 도저히 감당할 수 없었을 것이다. 그렇기에 타협을 한 것이다.

교재 저자는 자바 컬렉션은 전 세계 모든 상황을 커버해야 하니 이런 극단적인 타협을 했지만, 독자들이 직접 프로그램을 설계할 때는 절대 이 방식(호출하면 에러 나는 엉터리 인터페이스)을 따라 하지 말 것을 주장한다...






















