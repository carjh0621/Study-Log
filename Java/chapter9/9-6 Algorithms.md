
## Why Generic Algorithms?

교재는 컬렉션 내에서 가장 큰 값을 찾는 알고리즘을 예시로 든다. 제네릭 구조가 없다면, 데이터를 담는 자료구조의 형태에 따라 매번 다른 방식으로 코드를 작성해야한다. 즉 일반적인 배열, ArrayList, LinkedList 의 경우 유사한 점이 많지만, 허용되는 것들이 다르고 접근 방식이 다르다. 
그렇기에 어떤 자료구조냐에 따라서 다음과 같이 메서드를 일일이 따로 만들어야 한다.
- `static <T> T max(T[] a)` 
- `static <T> T max(ArrayList<T> v)` 
- `static <T> T max(LinkedList<T> l)`
이는 코드 중복과 자잘한 버그를 발생시키기 쉽다. 또한 테스트의 양도 많아진다.

따라서 제네릭 알고리즘이 도입되었다. 핵심은 이 알고리즘을 수행하기 위해 필요한 최소한의 기능이 무엇인지를 따지는 것이다. 
최댓값을 찾기 위해서는 처음부터 끝까지 요소들을 한번 훑을 수 있으면 충분하다. 
따라서 구체적인 클래스에 의존하는 대신, 훑어보기 기능을 보장하는 가장 상위의 추상적인 인터페이스인 `Collection`을 기준으로 코드를 작성하면 된다. 
교재의 max 메서드 예시를 보자
```java
public static <T> T max(Collection<T> c)
{
   //컬렉션이 비어있는지 먼저 확인하여 에러를 방지
   if (c.isEmpty()) throw new NoSuchElementException();
   
   Iterator<T> iter = c.iterator();
   
   // 첫 번째 요소를 임시 최댓값으로 
   T largest = iter.next();   

   while (iter.hasNext())
   {
      T next = iter.next();
      //현재 요소와 임시 최댓값을 비교하여 더 큰 값으로 업데이트
      if (largest.compareTo(next) < 0)
         largest = next;
   }
   //모든 순회가 끝나면 최종 최댓값을 반환
   return largest;
}
```
- `Collection<T> c` 로 인해, `LinkedList`든 `ArrayList`를 넘기든 상관이 없어진다. 




## Sorting and Shuffling

자바는 컬렉션 프레임워크 내에 내장 정렬기능을 제공한다. 리스트를 정렬하려면 `java.util.Collections` 클래스의 `sort` 메서드를 사용하면 된다.
: `Collections.sort`
- 리스트에 담긴 요소들이 Comparable 인터페이스를 구현한 객체일 때 사용한다. 기본 정렬이라고 할 수 있다. Comparable 인터페이스에 정의된 기본 오름차순 규칙에 따라 정렬된다.
: `List.sort`, `Comparator`
- 기본 규칙 이외에 다른 기준이 필요하다면, `java.util.List` 인터페이스에 정의된 `sort` 메서드와 함께, `Comparator`를 전달하면 된다.
- ex) `staff.sort(Comparator.comparingDouble(Employee::getSalary));` - 직원 리스트를 연봉 기준으로 정렬
: 역순 정렬
- `Comparator.reverseOrder()` 를 사용하거나 `.reserved()` 붙여 직접 만든 기준을 뒤집을 수 있다.

배열이 아닌 리스트의 정렬? (`LinkedList`)
: LinkedList의 경우 인덱스 기반 접근이 불가하므로 정렬 효율이 떨어지기 마련이다. 
- 자바의 경우 리스트의 모든 요소를 배열에 넣은 뒤 배열 상태에서 정렬을 하고 리스트에 복사하는 식으로 한다.

Stable Sort?
: 자바의 컬렉션 정렬은 Stable 정렬 알고리즘을 사용한다. (Quick-Sort x). stable함은 동일한 값을 가진 요소들의 순서가 정렬 후에도 그대로 유지되는 특성을 의미한다.
- 직원 리스트를 이름순으로 정렬하고 연봉순으로 정렬했다고 하자. 연봉이 똑같은 두 직원이 있다면, 이 둘은 이전에 정렬해 둔 이름순을 그대로 유지하게 된다. --> 복합 정렬이 성공함.

`sort` 알고리즘의 적용 대상?
: 모든 컬렉션이 sort 알고리즘을 거칠 수 있는것은 아니다. 예를들어, Unmodifiable List를 정렬하려면 에러가 난다. 
- Modifiable(set 메서드 지원), Resizable(add, remove 메서드 지원)
- 수정 가능해야하지만, 크기를 조절하는건 선택사항이다.

shuffling
: 리스트의 요소들을 무작위로 섞는 `Collections.shuffle` 메서드가 있다. 
- 시간 복잡도는 $O(n \times a(n))$ 이다. $a(n)$은 요소에 접근하는데 걸리는 평균시간이다.
- 이 메서드 역시 인덱스 기반의 빠른 접근이 불가능한 리스트가 들어오면(LinkedList), 배열로 복사해 섞은 후 다시 리스트로 돌려놓는 방식을 사용한다.


## Binary Search

Linear Search의 선형 시간복잡도와 달리 데이터가 이미 정렬되어 있다면, 로그시간 안에 이진탐색으로 끝낼 수 있다.
`java.util.Collections` 의 내장된 `binarySearch` 메서드를 사용하면 된다. 
- `int i = Collections.binarySearch(c,element);` = 리스트 c에서 element를 찾기
만약 기본 오름차순정렬이 아닌, 특정 기준으로 정렬된 리스트라면 탐색할 때도 동일한 `Comparator`를 넘겨주어야 한다.
- `int i = Collections.binarySearch(c, element, comparator);`

반환값과 삽입 위치
: `binarySearch`는 int를 반환한다. 반환된 숫자를 해석하는것이 중요하다.
- 반환값 >= 0
	- 반환된 값이 해당 데이터가 위치한 인덱스를 의미한다. 
- 반환값 < 0 
	- 단순히 데이터가 없다를 알려주는 것이 아닌, 이 데이터를 추가한다면 어디에 넣어야 정렬을 깨뜨리지 않을지에 대한 정보를 음수값으로 반환한다.
	- $insertionPoint = -i-1$

이진 탐색은 로그 시간을 만족하려면 Random Access가 가능해야 한다. 
만약 `LinkedList`같은 RandomAccess 인터페이스를 구현하고 있지 않은 리스트를 이진탐색에 넣으면 이진탐색을 포기하고 순차탐색을 하게된다.



## Simple Algorithms

코드를 직접 짜는 것보다 라이브러리를 사용하는 가장 큰 이유?
: 가독성 때문이다. 사람이 유지보수를 해야하므로, 결국 이 코드가 어떤 목적을 가지고 있는지를 한번에 파악하는 것이 중요하다. 

자바 8부터는 `Collections`유틸리티 클래스를 거치지 않고, 컬렉션 객체 자체가 스스로 유용한 연산을 수행할 수 있도록 Default Method가 추가되었다. 그렇기에 여기에 람다식을 넘기면 보다 압축해서 기능을 수행할 수 있다.
```java
// 길이가 3 이하인 단어들을 리스트에서 제거
words.removeIf(w -> w.length() <= 3);

// 리스트의 모든 단어들을 소문자로 변환해라
words.replaceAll(String::toLowerCase);
```

---
`Collection` vs `Collections`
- `java.util.Collection` : 데이터를 담는 자료구조(리스트, 셋, 큐)가 공통적으로 가져야할 인터페이스
	- add, remove, size, isEmpty 같은 메서드들이 선언됨.
- `java.util.Collections` : `Collection` 인터페이스를 구현한 객체들을 위한 Helper 클래스
	- 내부의 모든 메서드가 `static` 이고, 이 클래스를 객체로 생성할 수 없도록 되어있다.
	- 컬렉션을 외부에서 매개변수로 주면 대신 작업해준다.

---


## Bulk Operations

Bulk Operation = 일괄 연산 
: add, remove 등의 메서드는 데이터를 한번에 하나씩 다루었다. 하지만 실제로는 A리스트의 데이터를 B리스트에서 전부 지워줘. 등의 다량의 데이터를 한번에 다루는 작업이 많다. 
그렇기에 `java.util.Collection`인터페이스에는 파라미터로 다른 컬렉션을 받는 일괄 연산 메서드들이 기본적으로 정의되어 있다. 

대표적인 두 가지 일괄 연산 메서드를 보자.
- `coll1.removeAll(coll2);` : 차집합
- `coll1.retainAll(coll2);` : 교집합

활용 예시
1. 
```java
// 모든 컬렉션은 다른 컬렉션을 파라미터로 받아 그 데이터를 그대로 복사해오는 생성자를 가지고 있다. 즉 아래의 코드는 `firstSet`의 복사본을 만든다.
var result = new HashSet<>(firstSet);
result.retainAll(secondSet);
```

2. 뷰와 같이 사용
`Map`은 `Collection` 족보에 들어가지 않지만 `Map` 안의 데이터인 키-값들을 한번에 조작하고 싶을때, 일괄 연산을 쓸 수 있는 뷰를 활용한 꼼수가 있다.
회사 직원 정보가 담긴 `staffMap`가 있고, 퇴사자들의 ID만 모아둔 집합 `terminatedIDs`가 있다고 하자.
```java
staffMap.keySet().removeAll(terminatedIDs);
```
이렇게 하면 뷰와 연결된 원본 `staffMap`에서 퇴사자들의 정보가 일괄 삭제된다.

3. 이외에도 뷰의 또다른 형태인 Subrange를 이용해서 일괄연산을 할 수 있다.



## Converting between Collections and Arrays

현대의 자바를 사용한 개발에는 컬렉션을 주로 사용한다. 하지만 자바의 오래된 API들은 이 여전히 전통적인 배열을 매개변수로 요구하거나 반환하는 경우가 많다. 
따라서 최신 컬렉션 구조와 과거의 배열 구조 사이를 논리적인 오류 없이 오가는 변환 기술을 숙지해야한다.

배열 -> 컬렉션
: 자바 9부터 추가된 `java.util.List` 인터페이스의 팩토리 메서드인 `List.of()`를 사용하면 된다.
```java
String[] names = {"java", "c++", "python"};
List<String> staff = List.of(names);
```
- 단 `.of` 를 사용해서 생성된 리스트는 수정불가 상태라는 점을 놓쳐선 안된다.

컬렉션 -> 배열
: `java.util.Collection` 인터페이스에 정의된 `toArray()` 메서드를 사용한다. 하지만 여기서 많이들 런타임 에러를 겪곤 한다.
```java
String[] names = (String[]) staff.toArray(); //Error
```
- 기본 `toArray()` 메서드는 내부 요소가 무엇이든 상관없이, 메모리에 가장 최상위 타입인 `Object[]` 형태의 배열을 생성해서 반환한다. 자바는 태생이 `Object[]` 로 만들어진 객체를 하위 타입인 `String[]`으로 캐스팅하는 것을 금지하고 있기 때문에 에러가 발생한다.

: 해결책은 형변환을 나중에 하는 것이 아닌, `toArray()` 메서드에 처음부터 `String[]`타입의 배열을 만들도록 생성자 참조를 주는 것이다.
```java
String[] values = staff.toArray(String[]::new);
```
- 자바 11 이전에는 `new String[0]` 처럼 길이가 0인 빈 배열을 넘겨서 힌트를 제공하는 우회 방식을 썼다.


## Writing Your Own Algorithms

컬렉션을 파라미터로 받거나 반환하는 본인만의 메서드를 만들 때 중요한 것은 다형성을 얼마나 잘 활용하느냐에 있다. 

메서드의 파라미터를 작성한다고 해보자. 이때는 가능하면 상위 타입으로 지정하는 것이 옳다.
`public void processItems(타입<Item> items)` 
여기서 타입을 ArrayList, Collection, Iterable 각각으로 지정할때의 효과를 살펴보자.
- `ArrayList` : 이 메서드를 호출하는 쪽에서 `LinkedList`, `HashSet`등을 넘겨주려고 한다면 강제로 `ArrayList`에 옮겨 담은 후 넘겨야 하는 문제가 있다.'
- `Collection` : 데이터의 묶음 (Collection 인터페이스 구현 객체)이 필요하다면 Collection으로 지정하는 것도 좋은 선택이다.
- `Iterable` : 만약 메서드 내부에서 향상된 for문만 돌릴 거라면, Collection의 상위인 `Iterable`을 요구하는 것이 가장 좋다.

두번째로 반환형을 고민할 때 나중에 수정사항이 있을 경우를 생각하는 것이 좋다.
예를들어 다음의 메서드를 보자.
```java
public List<Item> lookupItems() {
   var result = new ArrayList<Item>();
   // ... 작업 수행 ...
   return result;
}
```
메서드 내부에서는 `ArrayList` 객체를 생성해서 쓰더라도, 반환형 자체는 인터페이스인 `List`로 선언하는 것이 좋다. 이렇게 하면 훗날 시스템을 최적화하기 위해 데이터가 없을 때는 굳이 무거운 `ArrayList`를 만들지 말고, 가벼운 `List.of()` 빈 리스트를 반환하는 것으로 내부 로직을 뜯어고치더라도, 외부에서 이 메서드를 호출하던 기존 코드는 전혀 망가지지 않기 때문이다.







