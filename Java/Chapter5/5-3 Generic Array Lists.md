
배열의 문제점: 크기를 처음에 정해야 함
Arraylist: 크기 미리 정하지 않음. 동적 배열

선언
- java 5+ : `ArrayList<Employee> staff = new ArrayList<Employee> ();`
- java 7+ : `ArrayList<Employee> staff = new ArrayList<> ();`
- java 10+ : `var staff = new ArrayList<Employee>();`
	- `var staff = new ArrayList<>()` 이러면 `ArrayList<Object>` 가 됨

동작 원리
- add(객체) 메서드로 데이터를 넣음
- 내부적으로는 배열을 가지고 있음
- 만약 내부 배열보다 많이 넣는다면
	- 더 큰 배열을 만들고 기존 데이터를 새 배열로 복사, 기존 배열 버림

Capacity, Size
- `new ArrayList<>(100)`
	- capacity가 100
	- size는 0
	- add로 채워 넣어야 사이즈가 늘어남

최적화
- 복사 비용이 큼
	- `ensureCapacity(int n)`
	- 미리 용량을 크게 바꾸는 것 --> 복사 최소화
- 남는 공간 절약
	- `trimToSize()`
	- 내부 배열 크기를 줄임


## Accessing Array List Elements

`[]` 대신 메서드
- 조회: `list.get(i)`
- 수정: `list.set(i,x)`

주의
- `set(index, value)` 는 이미 존재하는 칸의 값을 바꿀 때 씀
- `new ArrayList<>(100)` ->`list.set(0,"data")` : 에러
- 빈 리스트를 채울 때는 무조건 `add(data)` 사용

중간 삽입/삭제
- `add(n,e)` : n번째 인덱스에 e 삽입
- `remove(n)` : n번째 인덱스 요소 삭제
- 중간 삽입/삭제 --> shift 발생 --> 성능 안좋음

배열 변환
- `ArrayList`로 유연하게 데이터 수집 후 고정된 배열로 변환 가능
- `var array = new Type[list.size()];`
- `list.toArray(array);`

---
```java
private static void printLogs(ArrayList<String> list) { 
	int index = 0; 
	for (String log : list) { 
		System.out.println(index++ + ": " + log); 
	} 
}
// 접근 방법
```
---


## Compatibility between Typed and Raw Array lists

옛날 코드와 요즘 코드를 섞어 쓸 때 벌어지는 일.

Raw Type
- `ArrayList list = new ArrayList();`
- `<Employee>` 등이 없음. 즉, `Object`를 담는 리스트.
- 옛날 코드

Typed -> Raw
- `ArrayList<Employee>` 를 만들어서, 파라미터로 `ArrayList`를 받는 옛날 메서드 `update(ArrayList list)` 에 넘김.
- 에러 x
- 위험성
	- 타입 검사 x 
	- 컴파일 에러 x, 다만 중간에 다루는 와중에 에러 발생 가능

Raw -> Typed 
- 옛날 메서드가 `ArrayList`를 반환, 이를 `ArrayList<Employee>` 에 넣으려는 상황
- 경고 발생(`Unchecked assignment`)
- 컴파일러 입장에서는 Object 리스트 안에 제대로 된 것이 들어있는지 확신 x

Casting?
- `(ArrayList<Employee>)` 로 강제 캐스팅 해도 경고는 사라지지 않는다.
- Type Erasure(소거)
	- 자바의 제네릭은 컴파일 할 때만 존재. 컴파일 끝나고 JVM에 올라갈 때(실행 중) 지워진다.
	- 즉, JVM 입장에서는 `ArrayList<...>` 모두 똑같은 `ArrayList`
	- 런타임에 내용물의 타입을 검사할 방법이 없다.

해결책
- `@SuppressWarnings("unchecked")` 어노테이션
- -> 개발자가 확인하고 컴파일러의 경고를 끔

