`Set`에서 어떤 테이터를 빠르게 찾으려면 찾으려는 그 데이터와 똑같은 객체 복사본이 필요하다.
하지만 실제로는 <xx번 사원의 이름> 처럼, 단서를 가지고 전체 정보를 찾아야 하는 경우가 대부분이다.
그래서 key-value를 쌍으로 묶어서 저장하는 자료구조가 필요하다.

## Basic Map Operations

`Map` 인터페이스를 구현하는 것들 중 유명한 것은 `HashMap`과 `TreeMap`이 있다. `HashSet`과 `TreeSet`과 원리가 똑같다. 다만 다른 것은 정렬과 해싱의 대상은 오직 Key 라는 것이다. 
- `HashMap` : Key의 hashcode를 계산해서 버킷을 찾는다. value는 해싱되지 않는다.
- `TreeMap` : Key를 기준으로 이진 검색 트리를 만들어 정렬한다. value는 정렬에 관여하지 않는다.
중요한 것은 Key는 중복될 수 없다는 것이다.

APIs
-데이터 추가 : `V put(K key, V value)`
```java
staff.put("987-98-9996", new Employee("Harry Hacker"));
```
- put 메서드는 덮어쓴 기존 value를 반환한다. 처음 넣는 거라면 `null`을 반환한다.

-데이터 조회: `V get(Object key)`, `getOrDefault`
```java
//일반적인 조회
Employee e = staff.get("987-98-9996"); // 찾으면 객체 반환, 없으면 null 반환

//Java 8+
int score = scores.getOrDefault(id, 0);
```
- 혹시 모를 `NullPointerException`을 예방하기 위해서 `getOrDefault`를 사용한다. id로 찾아보고 없으면 0(기본값)을 반환하는 메서드이다.

-데이터 순회: `forEach`
```java
staff.forEach((k, v) -> System.out.println("key=" + k + ", value=" + v));
```
- `Map`은 `Collection` 인터페이스의 자식이 아니다. 따라서 향상된 for문을 사용할 수 없다.
- 자바 8부터 `BiConsumer` 를 활용한 `forEach` 메서드가 도입되어 람다식으로 key, value를 뽑아 쓸 수 있다.

데이터 삽입 시에는 제네릭 타입 `K`를 쓴다.  하지만 데이터를 찾거나 지울 때는 `Object`를 쓴다. 
해시 테이블에서 데이터를 찾을 때는 오직 `hashCode()`와 `equals()` 메서드만 사용한다. 이 두 메서드는 `Object`클래스에 정의되어 있다. 
따라서 굳이 `K`와 일치하지 않더라도 논리적으로 `equals()`를 통해 같다고 판별되는 다른 타입의 객체를 사용해서 데이터를 찾을 수 있도록 가능성을 열어둔 것이다. 
논란이 좀 있을지도...??


## Updating Map Entries

텍스트 파일에서 어떤 단어가 몇 번 나왔는지 세는 알고리즘(word count)을 만든다고 생각해보자. 단어를 key, 나온 횟수를 value로 저장해야 한다.
다음과 같이 설계한다고 해보자
```java
counts.put(word, counts.get(word)+1);
```
- 문제는 이 단어가 처음 등장했을 때이다. `get(word)`는 맵에 없는 키를 찾게 돼서 `null`을 반환한다. `null + 1`을 계산하려다 `NullPointerException` 예외가 던져진다.

해결책
-1. `getOrDefault`
```java
counts.put(word, counts.getOrDefault(word, 0) + 1);
```
-2. `putIfAbsent`
```java
counts.putIfAbsent(word, 0); // 맵에 이 단어가 아예 없을 때만 0을 삽입
counts.put(word, counts.get(word) + 1); // 이제 데이터가 있으니 문제가 없다
```
-3. `merge`
```java
counts.merge(word, 1, Integer::sum);
```
- word: 키
- 1 : 처음 삽입할 경우 넣을 초기값
- `Integer::sum` : 이미 값이 존재할 경우, 기존 값과 새로운 값을 어떻게 병합할 것인지 정의한 람다식
- ==> 맵에 word가 없으면 1을 넣고, 이미 숫자가 있으면 기존 숫자랑 1을 sum 해서 그 결과로 덮어씌워라.

유용한 API: `computeIfAbsent` 
: 부서 이름을 주면 해당 부서의 직원 리스트를 관리하는 맵(`Map<String,List<Employee>)`)이 있다고 해보자. 처음 직원을 넣을 때는 빈 리스트부터 새로 만들어야 한다.
```java
staffListByDept.computeIfAbsent("개발팀", k -> new ArrayList<>()).add(harry);
```
개발팀 리스트가 없으면 우측으 람다식을 실행해서 `new ArrayList`를 빈 공간에 만들고, 그 리스트를 반환하면 거기에 직원을 추가하는 코드이다.


## Map Views

`Map`은 `Collection` 인터페이스의 자식이 아니다. 따라서 향상된 for문을 직접 돌릴 수 없다. 
하지만 맵 안에 있는 키만 싹 다 뽑아서 리스트로 보고 싶거나, 값들만 모아서 순회하고 싶다는 요구들이 있기 마련이다.

자바는 3개의 view를 제공한다.
- `Set<K> keySet()` : key들만 모아서 보여줌 (중복 x 라 `Set`으로 반환)
- `Collection<V> values()` : 값만 모아서 보여줌. (값은 중복 가능이므로 `Collection`으로 반환)
- `Set<Map.Entry<K, V>> entrySet()` : 키와 값의 쌍을 하나의 Entry로 묶어서 보여줌

뷰 객체들을 사용할 때 생각해야 하는것이 있다. 
`keySet()`, `values()` 가 반환하는 객체는 맵의 데이터를 복사해서 새로 만든 컬렉션이 아니다. 실제 맵과 연결된 것이다. (Live view)
고로 삭제는 허용된다.
```java
Set<String> keys = map.keySet();
keys.remove("987-98-9996"); // view에서 키를 지운다. 그러면 원본에서도 지워짐
```
그리고 추가는 불가하다.
```java
keys.add("123-45-6789"); // ERROR!
```
- 키만 주고 value는 뭔지 알려주지 않은 채로 원본에 추가해주면 모순이다.



## Weak Hash Maps

특정 객체에 대한 메타데이터를 맵에 저장해 두는 경우가 많다. 예를들어 어떤 `User` 객체를 키로 삼아 `Session` 정보를 값으로 저장해 두었다고 가정해보자. 시간이 지나 사용자가 로그아웃하면 프로그램의 다른 곳에서는 더 이상 이 `User` 객체를 사용하지 않는다. 
상식적으로 안 쓰는 객체는 가비지 컬렉터(GC)가 메모리에서 지워야한다. 하지만 `HashMap`이 해당 `User` 객체를 키로 잡고 있기때문에 GC는 아직 맵이 쓰고 있다고 착각하여 메모리에서 지우지 못한다. 이것이 치명적인 메모리 누수이다. (Strong Reference)

이 문제를 해결하기 위해 나온 것이 `WeakHashMap`이다. 이는 Weak Reference라는 특수한 방식으로 키를 가진다. 즉 언제든 키를 놓을 수 있다.
GC와 어떤 식으로 상호작용하는지 보자.
1. GC의 순찰: 가비지 컬렉터가 메모리를 청소하러 돌아다님.
2. 약한 참조 발견: 특정 키 객체를 보니, 프로그램 내의 다른 곳에서는 아무도 안 쓰고 오직 `WeakHashMap` 내부에서만(그것도 약한 참조로) 들고 있는 것을 발견한다
3. 삭제: GC는 메모리에서 키 객체를 지운다.
4. 사후 처리 (ReferenceQueue): GC는 키를 지운 후, `WeakHashMap`이 가지고 있는 특수한 큐(Queue)에 기록을 남긴다
5. 맵의 자가 정비: 맵이 메서드(`get` 등)를 호출받아 동작할 때마다 이 큐를 확인한다. 삭제된 기록이 있으면, 키가 지워져 버린 쓸모없는 값도 맵에서 삭제하여 메모리를 회수한다.



## Linked Hash Sets and Maps

