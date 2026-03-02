
자바의 자료구조는 두 가지로 나눌 수 있다.
`Collection` : 데이터를 그 자체로 담으며 앞에 나온 `add`를 사용하고 `Iterator`로 순회한다.
`Map` : 데이터를 key-value 쌍으로 저장한다. (사전같은 구조)
- `put(K key, V value);`, `get(Object key);`
- `Map`은 `Collection`인터페이스를 상속받지 않는다.

`Collection`을 상속받는 가장 대표적인 두 자식 인터페이스를 살펴보자
: 1. `List`
- 데이터가 입력된 순서를 기억하며, 중복된 데이터를 허용하는 자료구조
```java
void add(int index, E element); // 특정 위치(index)에 데이터를 삽입
void remove(int index);         // 특정 위치의 데이터를 삭제
E get(int index);               // 특정 위치의 데이터를 읽음
E set(int index, E element);    // 특정 위치의 데이터를 덮어씀.
```
- `ListIterator` : `List` 전용 반복자. 일반 `Iterator`와 달리 뒤로 갈수 있고 (`previous`) 현재 커서 위치에 데이터를 바로 추가 가능하다.
: 2. `Set`
- 중복을 허용하지 않는 데이터의 집합.
- `Collection` 인터페이스와 메서드 시그니처가 똑같다. 새로운 메서드도 없다.
- 다만 규칙이 엄격해진다.
	- `add` 메서드는 이미 존재하는 데이터를 넣으려 하면 무시하고 `false`를 반환한다.
	- `equals` 메서드는 데이터의 순서가 달라도 내부 요소만 똑같으면 두 `Set`이 같다고 판별한다


`Sorted`, `Navigable` 인터페이스
- `Set`, `Map` 안의 데이터들을 항상 자동 정렬해주는 기능

교재 저자는 이 부분이 형편없이 설계되었다고 비판한다.
: `List` 인터페이스에는 `get(int index)`라는 임의 접근 메서드가 강제되어 있다. 이는 배열 기반의 리스트에는 적합하지만, 연결 리스트의 경우에는 순차적으로 접근해야한다.($O(n)$) 즉, 애초에 임의 접근용 인터페이스와 순차 접근용 인터페이스를 따로 만들어야 한다는 것이다.

자바 설계자들은 자바1.4에서 이 실수를 수습하기 위해 `RandomAccess`라는 메서드가 없는(Tagging, Marking 인터페이스) 인터페이스를 만들었다.
: 이는 인덱스 접근을 해도 속도가 빠른 클래스를 나타내는 역할을 한다. `ArrayList`는 이 이름표를 달고 있고, `LinkedList`는 달고 있지 않는다.
: 불특정 `List`를 받아 데이터를 처리할 때는 아래와 같이 성능을 최적화하는 분기 처리를 해야한다.
```java
// 전달받은 컬렉션 c가 어떤 내부 구조인지 모를 때
if (c instanceof RandomAccess) {
    for (int i = 0; i < c.size(); i++) {
        Object o = c.get(i); // 빠른 임의 접근 사용
    }
} else {
    for (Object o : c) { 
        // Iterator를 활용한 순차 접근 사용 (향상된 for문)
    }
}
```


