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

---
교재의 view 관련된 설명을 보고 헷갈리면 안되는 것이, view 객체가 실제로 맵의 키, 값 등을 실제로 참조하고 있다고 착각할 수 있다는 것이다.
예를 들어, 기존 맵에 100만개의 키-값 쌍이 있다고 하자. 그리고 keySet()을 호출한다고 하면, 100만개의 데이터에 대해서 참조를 모두 생성하는 것이 아니다. 
실제로는 기존 맵에 대한 주소만 저장(참조)하고 view를 통한 어떤 접근이 들어오면, 그걸 기존 맵에 넘기는 식으로 작동한다. 
즉, `keySet()`이 반환하는 것은 키들의 묶음이 아니라, 원본 Map을 Set의 규격(인터페이스)에 맞춰서 조종할 수 있게 해주는 리모컨? 을 반환한다고 보면 된다. (contains, remove를 오버라이드 한 것)

---



## Weak Hash Maps

특정 객체에 대한 메타데이터를 맵에 저장해 두는 경우가 많다. 예를들어 어떤 `User` 객체를 키로 삼아 `Session` 정보를 값으로 저장해 두었다고 가정해보자. 시간이 지나 사용자가 로그아웃하면 프로그램의 다른 곳에서는 더 이상 이 `User` 객체를 사용하지 않는다. 
상식적으로 안 쓰는 객체는 가비지 컬렉터(GC)가 메모리에서 지워야한다. 하지만 `HashMap`이 해당 `User` 객체를 키로 잡고 있기때문에 GC는 아직 맵이 쓰고 있다고 착각하여 메모리에서 지우지 못한다. 이것이 치명적인 메모리 누수이다. (Strong Reference)

이 문제를 해결하기 위해 나온 것이 `WeakHashMap`이다. 이는 Weak Reference라는 특수한 방식으로 키를 가진다. 즉 언제든 키를 놓을 수 있다.
GC와 어떤 식으로 상호작용하는지 보자.
1. 가비지 컬렉터가 메모리를 청소하러 돌아다님.
2. 특정 키 객체를 보니, 프로그램 내의 다른 곳에서는 아무도 안 쓰고 오직 `WeakHashMap` 내부에서만(그것도 약한 참조로) 들고 있는 것을 발견한다
3. GC는 메모리에서 키 객체를 지운다.
4. (ReferenceQueue): GC는 키를 지운 후, `WeakHashMap`이 가지고 있는 특수한 큐(Queue)에 기록을 남긴다
5. 맵이 메서드(`get` 등)를 호출받아 동작할 때마다 이 큐를 확인한다. 삭제된 기록이 있으면, 키가 지워져 버린 쓸모없는 값도 맵에서 삭제하여 메모리를 회수한다.

---
**Strong Reference**
```java
User key = new User("Harry");
```
- 평소에 쓰는 `=` 할당 연산자. 
- 스택에서 힙에 있는 `User` 객체의 메모리 주소를 가리킴
- GC는 객체가 어디서도 참조되지 않을때 지운다.

**Weak Reference**
```java
WeakReference<User> weakKey = new WeakReference<>(new User("Harry"));
```
- 약한 참조는 `java.lang.ref.WeakReference`라는 Wrapper 클래스의 객체이다. 즉 스택이 `WeakReference` 객체를 참조하고 이 객체 안에 타켓인 `User` 객체가 있는 구조이다.
- GC는 타겟을 참조하는 것이 WeakReference 객체뿐이면 타겟을 메모리에서 삭제한다.
- 여기서 생각치 못한 문제가 여전히 발생 가능하다.
	- 예를들어 키가 GC에 의해서 삭제된 후 `WeakHashMap`의 메서드를 호출하지 않는다고 해보자.
	- 이러면 이 맵 안의 큐를 확인할 기회가 오지 않으므로 삭제된 키에 해당하는 value 객체는 메모리에 계속 남아 memory leak이 일어난다.
	- 그렇기에 `WeakHashMap`은 주기적으로 접근이 일어나는 캐시환경에서만 안전하다.

---


## Linked Hash Sets and Maps

`HashMap`은 해시 함수에 의해 무작위 버킷에 들어가기 때문에 입력된 순서를 완전히 잃는다는 단점이 있다. 

`LinkedHashMap`은 해시테이블의 상수시간 속도를 유지하면서도 데이터들을 이중연결 리스트로 다시 한번 엮어서 순서를 기억하게 만든 자료구조이다.
![[Pasted image 20260224150433.png]]


`LinkedHashMap` 은 생성자 설정에 따라 두가지 순서 모드중 하나로 작동한다.
- 기본값은 삽입 순서대로 이어 붙이는 것이다. 출력하면 항상 넣은 순서대로 나온다.
- 생성자의 마지막 인자를 `true`로 하면 (`new LinkedHashMap<>(128, 0.75F, true)`), 데이터를 읽거나 덮어쓸 때마다 맵은 그 데이터를 기존 위치에서 뽑아내어 연결리스트의 맨 끝으로 이동시킨다. 즉, 리스트의 맨 앞에는 항상 가장 오랫동안 찾지 않은 데이터가 남게 된다. 이는 LRU 와 동일한 동작을 한다.

LRU 캐시를 `LinkedHashMap`으로 만드는 방법.
```java
var cache = new LinkedHashMap<K,V>(128,0.75F,true){
	@override
	protected boolean removeEldestEntry(Map.Entry<K,V> eldest){
			return size()>100;
		}
}
```
- 교재에 나와있는 이 코드를 보면, 데이터가 들어올 때마다 위의 메서드가 자동으로 호출된다. 그때 맵의 현재 사이즈가 100을 넘으면 true를 반환한다. true가 반환되면 맵은 알아서 가장 오래된 데이터를 삭제한다. 
- 여기서 LinkedHashMap의 마지막 인자를 false로 하면 그냥 현재 데이터들 중 가장 먼저 추가된 데이터를 의미하게된다.



## Enumeration Sets and Maps

자바에서 `Enum`은 미리 정해진 유한한 개수의 상수들만 가질 수 있는 타입이다. 이처럼 값의 개수가 고정되어 있다는 특징을 활용하면 데이터를 빠르고 효율적으로 다룰 수 있다.

일반적인 `HashSet`을 써도 열거형 데이터를 담을 수 있지만, `EnumSet`이라는 클래스를 별도로 제공한다. (훨씬 빠름)
- 열거형은 인스턴스 개수가 제한적이다. 따라서 `EnumSet`은 내부적으로 복잡한 자료구조 대신 비트의 나열로 구현된다. (비트마스킹) . 이 방식은 메모리를 적게 차지하여 처리 속도가 매우 빠르다.
- `EnumSet`은 `new EnumSet()` 과 같은 public constructor를 제공하지 않는다. 대신 정적 팩토리 메서드를 사용해야 한다.

교재의 예시를 따라가보자
	요일을 나타내는 열거형 `Weekday`가 있다고 가정해보자
```java
enum Weekday { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY };
```
아래는 정적 팩토리 메서드를 이용해 다양한 상태의 `EnumSet`을 만드는 방법들이다.
-  `allOf` : 해당 열거형의 모든 값을 포함하는 집합을 만든다.(모든 비트 1)
```java
    EnumSet<Weekday> always = EnumSet.allOf(Weekday.class);
```
- **`noneOf`**: 해당 열거형을 담을 수 있는 빈 집합 (모든 비트를 0으로 끔)
``` java
    EnumSet<Weekday> never = EnumSet.noneOf(Weekday.class);
```
- **`range`**: 시작점과 끝점을 지정해 특정 범위의 값들만 포함하는 집합을 만듦. (월~금 등)
``` java
    EnumSet<Weekday> workday = EnumSet.range(Weekday.MONDAY, Weekday.FRIDAY);
```
- **`of`**: 원하는 특정 값들만 골라서 집합을 만듦
```java
    EnumSet<Weekday> mwf = EnumSet.of(Weekday.MONDAY, Weekday.WEDNESDAY, Weekday.FRIDAY);
```
이렇게 만들어진 `EnumSet`은 일반적인 `Set`과 동일하게 `add()`, `remove()`, `contains()` 등의 기존 메서드를 모두 정상적으로 사용할 수 있다

`EnumMap`도 있다. 이는 키에 열거형 타입만 올 수 있도록 된 맵이다. 이 역시 내부 원리가 단순하고 효율적이다. 열거형에 정의된 상수들은 내부적으로 0부터 시작하는 순서를 가진다. `EnumMap`은 이 순서를 배열의 인덱스로 사용하여 데이터를 저장한다. 즉 해시 계산 없이 접근하므로 `HashMap`보다 검색 속도가 빠르다.

`EnumMap`은 `EnumSet`과 달리 `new` 키워드를 사용해 생성자를 호출한다. 다만 생성자의 매개변수로 어떤 열거형을 키로 쓸 것인지 클래스타입을 명시해주어야 한다.(배열의 크리를 미리 알아야 함)
```java
var personInCharge = new EnumMap<Weekday, Employee>(Weekday.class);
//요일별 직원 
```



## Identity Hash Maps

일반적인 `HashMap`은 키를 비교할 때 `equals()`, `hashCode()` 메서드를 사용한다. 이는 두 객체가 메모리상에서 서로 다른 곳에 존재하더라도 값이 같으면 같은 키로 취급하여 데이터를 덮어씌운다.
하지만 프로그래밍을 하다보면 내용이 같더라도 물리적으로 완전히 동일한 객체가 아니면 다른 키로 취급해야하는 특수한 상황이 발생한다.
이때 쓸 수 있는것이 `IdentityHashMap`이다. 이 클래스는 키를 비교할 때, `==` 연산자를 사용한다.(메모리 주소를 직접 비교). 또한 해시값을 계산할 때도 메모리 주소 기반으로 해시값을 강제 생성해주는 `System.identityHashCode(Object)`메서드를 사용한다.
- 객체 직렬화나 객체 그래프 탐색 알고리즘에 사용된다. 예를 들어 방문했는지 살필 때, 우연히 내용이 같은 경우를 예방할 수 있다.





























