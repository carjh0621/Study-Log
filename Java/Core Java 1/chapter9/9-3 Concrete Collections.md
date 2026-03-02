## Linked List
`ArrayList` -> 메모리가 연속적이라 끝에 데이터를 넣는 것은 괜찮지만, 중간에 삽입하거나 지울 경우 데이터를 이동(shift) 해야하므로 $O(n)$ 의 비용이 든다.

그래서 데이터가 메모리상에 흩어져 있어도 참조를 통해 이어주는 연결 리스트가 등장하게된다.
자바의 `LinkedList` 는 이중 연결 리스트이다.

자바는 `Iterator`를 확장한 `ListIterator`를 제공한다. 이는 앞뒤로 움직이며 데이터를 추가,수정 할 수 있다.
: `add(E element)`
```java
ListIterator<String> iter = staff.listIterator();
iter.next(); // 첫 번째 요소를 건너뜀 (커서가 1번과 2번 사이에 위치함)
iter.add("Juliet"); // 커서가 있는 현재 위치에 "Juliet"을 끼워 넣음
```
: `remove()` , `set(E)`
- 방금 next()나 previous()로 커서로 지나간 바로 그 데이터를 지우거나 교체
: Fail-Fast 메커니즘
```java
ListIterator<String> iter1 = list.listIterator();
ListIterator<String> iter2 = list.listIterator();
iter1.next(); iter1.remove(); // iter1이 리스트 구조를 바꿈
iter2.next(); // ERROR! iter2가 예외를 던진다
```
- 반복자들은 자기가 순회를 시작할 때의 리스트 상태를 기억한다. 만약 누군가가 구조를 바꾼다면, 길을 잃어버릴 수 있기 때문에 예외를 발생한 후 죽는다. (`ConcurrentModificationException`)

---
Java21에서 컬렉션 프레임워크 업데이트가 있었음
- 이전에 `LinkedList`에서 첫 번째 요소를 가져오려면 `getFirst()` 사용했지만, `ArrayList`는 `get(0)`을 써야했다. -> 일관된 표준 x
- 자바 설계자들은 `SequencedCollection`이라는 새로운 인터페이스를 만듦. list, deque, linkedlist, arraylist 등이 이를 상속받도록 함
- 이제 컬렉션 종류에 상관없이 `getFirst()`, `getLast()`, `addFirst()`, `addLast()`, `reversed()`(역순 뷰 반환) 메서드를 완벽하게 통일된 방식으로 사용할 수 있게 됨.

리스트 상태를 기억?
: `modCount` : `LinkedList`, `ArrayList` 같은 컬렉션 객체 내부에 존재하는 정수형 변수. 데이터를 추가하거나 삭제할 때마다 이 숫자가 1씩 증가.
: `expectedModCount` : iterator() 를 호출하여 반복자를 생성하는 순간 반복자는 컬렉션 본체의 현재 modCount 값을 expectedModCount에 저장한다. 
- `modCount`와 `expectedModCount`가 다르면 예외를 던지는 것

---


## Array Lists

배열과 유사하지만, 내부적으로 꽉 차면 스스로 더 큰 배열을 만들고 이사가는 배열

과거에는 `Vector`라는 클래스가 이 역할을 함. 하지만 `Vector`의 모든 메서드는 멀티스레드 환경에서 안전하게 동작하도록 락을 건다. 하지만 대부분의 경우 단일스레드에서만 쓰이므로 락 오버헤드가 두드러진다.  `ArrayList`는 이 동기화를 제거한 버전이다. 

## Hash Sets

해시 테이블이 데이터를 저장하고 찾는 순서는 다음과 같다.
- 1. Hash Code 생성: 객체의 데이터를 고유한 정수로 변환한다.
- 2. Bucket 배정: 이 숫자를 버킷의 크기로 나눈 나머지를 구한다. 
	- 해시값이 76268, 버킷이 128개면 -> $76268 (mod 128) = 108$. 즉, 108번 버킷에 데이터를 저장한다.
- 3. 나중에 찾을 때 해시값을 구해 상수 시간안에 찾는다.

Hash Collision
: 만약 서로 다른 데이터가 우연히 같은 인덱스를 부여받으면, 그 버킷에 연결 리스트를 둔다.
: 버킷이 꽉 차서 충돌이 너무 자주 일어나면 연결 리스트가 길어져 검색 속도가 느려진다. 그래서 Load Factor (적재율, 데이터/전체 버킷)가 75%가 되면, 버킷의 개수를 2배로 늘리고 모든 데이터를 재배치 (Rehashing)한다. 

`HashSet`이 제대로 동작하기 위해 지켜야 할 규칙이 있다.
- 만약 `a.equals(b)`가 `true`라면 (두 객체가 논리적으로 같다면), 반드시 `a.hashCode() == b.hashCode()`도 `true`여야 한다. (같은 방에 들어가야 찾을 수 있기 때문)
- 불변성 경고 (Caution): 교재 마지막의 경고가 매우 중요하다. `HashSet`에 객체를 넣은 후, 그 객체의 내부 데이터를 수정해서 해시 코드가 바뀌어버리면 절대 안 된다. 이미 108번 방에 들어갔는데 나중에 계산식이 바뀌어 50번 방 주소가 나오면, 영원히 그 데이터를 찾을 수 없게 된다. (미아 발생)

Hash Collision Dos
: 악의적인 해커가 고의로 해시 코드가 똑같이 나오는 데이터만 수만 개를 서버에 던진다면? 특정 버킷 하나에만 연결 리스트가 수만 개 매달리게 되어, 서버의 검색 속도가 $O(1)$에서 $O(n)$으로 저하되고 서버가 뻗는다.
: Java 8의 해결책
- 하나의 버킷(방)에 데이터가 8개 이상 뭉치면, 자바는 내부적으로 연결리스트를 Red-Black Tree로 자료구조를 바꾼다. 이렇게 하면 공격이 들어와도 탐색 속도를 $O(\log n)$으로 방어할 수 있다.


## Tree Sets

만약 학생들의 점수나 알파벳 순서대로 목록을 뽑아야 하는 상황에서 `HashSet`이나 `ArrayList`를 쓰면, 데이터를 다 넣은 후에 `Collections.sort()`를 따로 호출해야한다.

차라리 데이터를 집어넣는 그 순간(add)마다 알아서 자기 자리를 찾아 들어가게 만들는 것이 더 효율적이다. 그러면 나중에 데이터를 꺼낼 때(Iterator)는 그냥 순서대로 뽑기만 하면 된다. 이 아이디어를 구현한 것이 `TreeSet`이다.
이는 내부적으로 레드블랙 트리를 사용한다.

트리가 왼쪽으로 갈지 오른쪽으로 갈지 여부를 결정하려면 두 요소를 비교할 수 있는 명확한 기준이 있어야 한다. 교재에는 두가지 방법을 제시한다.
: `Comparable` 인터페이스
- 객체 자체가 원래부터 비교 기준을 가지고 있는 경우이다.
- 클래스가 `Comparable<T>`를 상속받아 `compareTo` 메서드를 오버라이딩한다.
```java
public int compareTo(Item other) {
    int diff = Integer.compare(partNumber, other.partNumber);
    // 번호가 같으면, 이름(description)을 사전순으로 비교
    return diff != 0 ? diff : description.compareTo(other.description);
}
// return 값이 음수면 내 자리가 왼쪽, 양수면 오른쪽, 0이면 중복으로 판별
```

: `Comparator` 인터페이스
- 객체의 원래 기준을 무시하고 새로운 기준을 사용하고 싶을 때 사용.
- `TreeSet`을 생성할 때, 생성자에 `Comparator` 객체를 전달한다.
```java
// Item 객체를 description 기준으로 비교하는 TreeSet 생성
var sortByDescription = new TreeSet<>(Comparator.comparing(Item::getDescription));
```



---
### ****`var sortByDescription = new TreeSet<>(Comparator.comparing(Item::getDescription));` 문법 설명을 해보자

자바 7부터 우항의 제네릭 타입을 생략할 수 있다. 컴파일러가 좌항을 보고 `<Item>`이 들어갈 것을 유추한다.
자바 10부터는 우항에 `new TreeSet`이 있는 것을 보고 좌항의 타입 전체(`var`)를 컴파일러가 알아서 결정한다.

`Item::getDescription` -> 메서드 참조. comparing은 키 추출기를 받는다. 넘겨준 메서드 참조는 내부적으로 Item을 넣으면 String을 밷어내는 함수인 `Function<Item, String>` 타입의 객체로 취급된다.

`Comparator.comparing(...)`
: 자바 8부터는 인터페이스 안에 `static`메서드를 만들 수 있다. `comparing`은 `Comparator`인터페이스가 제공하는 정적 팩토리 메서드이다. 
: 전달받은 추출기를 사용해, 새로운 `Comparator` 객체를 생성해 반환한다.
```java
// Comparator.comparing 내부에서 논리적으로 일어나는 일 (실제 자바 소스코드와 거의 동일)
public static <T, U> Comparator<T> comparing(Function<T, U> keyExtractor) {
    
    // T(Item) 두 개가 들어오면, 아까 받은 추출기로 각각 U(String)를 뽑아낸 뒤 비교하는
    // 규칙을 가진 새로운 Comparator 객체를 생성해서 반환한다.
    return (T c1, T c2) -> {
        U key1 = keyExtractor.apply(c1); // 첫 번째 Item의 description 추출
        U key2 = keyExtractor.apply(c2); // 두 번째 Item의 description 추출
        
        return key1.compareTo(key2);     
        // 두 String의 알파벳 순서를 비교하여 int 반환 (-1, 0, 1)
    };
}
```
메서드를 반환할 때 컴파일러가 이 람다식을 감싸는 익명 클래스의 인스턴스를 자동으로 생성한다.


`TreeSet`은 이 객체를 유지한다.
```java
public class TreeSet<E> {
    private Comparator<? super E> comparator; 

    public TreeSet(Comparator<? super E> comparator) {
        this.comparator = comparator; // 전달받은 감별사를 내부에 셋팅 
    }
    // ...
}
```
이후 sortByDescription.add 를 호출하면 내부의 comparator를 사용한다.
또한 `? super Item`이므로 `Comparator<Item>`은 `Item` 자신을 만족하므로 문제없이 참조할 수 있다.

`new TreeSet<>()` 처럼 Comparator를 넘겨주지 않으면 null이된다. 이때는 item의 compareTo를 사용한다.

---



## Queues and Deques

`Queue` : 데이터는 항상 Tail로 들어와서 Head로 나간다.
`Deque` : Double-Ended Queue, Head와 Tail 모두에 입출력이 가능하다

교재의 API 노트를 보면 추가, 삭제, 조회를 하는 메서드가 한 쌍으로 존재한다.
이는 12장에 나올 Bounded Queue 때문이다. 큐가 꽉 차있거나 비었을때 실패를 어떻게 처리할 것인가에 대한 철학의 차이이다.
- `add(e)` 는 꽉 찬 경우 `IllegalStateException`을  `remove()`, `element()` 는 `NoSuchElementException`을 던진다.
- `offer(e)` 는 꽉 찬 경우 예외 대신 `false`를 `poll()`, `peek()` 는 빈 경우 `null`을 반환한다.
	- 대기열 시스템의 경우 서버가 죽으면 안되므로 보통 후자를 많이 쓴다.

자바 1.0에는 `Stack`라는 클래스가 있었다. 하지만 `Vector`를 상속받아 느리다. 따라서 스택이 필요하면 `ArrayDeque`를 생성해서 `push()`와 `pop()`을 사용하는 것이 권장된다. `ArrayDeque`는 스레드 안정성은 없지만 가장 빠른 큐, 스택 구현체이다.


## Priority Queues

모든 데이터를 완벽하게 정렬할 필요가 없다. 오직 우선순위가 가장 높은 것만 바로 추출할 수 있으면 된다. 
여기에는 힙이라는 이진 트리 구조를 사용한다. 힙은 부모 노드는 항상 자식 노드보다 우선순위가 높다. 하지만 형제 노드끼리는 누가 큰지 신경쓰지 않는다.
- 따라서 Iterator로 순회가 가능해도 출력 결과가 정렬되어있지는 않다는 점에 유의해야한다.
새로운 값을 추가하거나 최우선순위 값을 뽑을때 트리의 한가지를 따라가면서 힙 속성을 유지하면 되므로 $O(logN)$ 이다.



