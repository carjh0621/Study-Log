`Manager`는 `Employee`의 자식이다. 그렇다면, 이들을 담고 있는 제네릭 클래스끼리의 관계는 어떻까?

흔히 Manager가 Employee 니까, `Pair<Manager>` 도 `Pair<Employee>`의 자식이라고 생각하기 쉽다. 하지만 실제로는 아니다. 
제네릭 타입 간에는 상속 관계가 성립하지 않는다. 

만약 이것이 허용된다면 어떤 문제가 발생하는지 보자
: 1. `managerBuddies` (관리자만 담는 쌍)를 `employeeBuddies` (직원 아무나 담는 쌍) 에 담을 수 있다고 치자
```java
Manager ceo = ...; 
Manager cfo = ...; 
Pair<Manager> managerBuddies = new Pair<>(ceo, cfo);

Pair<Employee> employeeBuddies = managerBuddies;
```

: 2. `employeeBuddies` 는 `Pair<Employee>` 타입이므로 당연히 일반 직원을 담을 수 있어야 한다.
```java
Employee lowlyEmployee = ...;
employeeBuddies.setFirst(lowlyEmployee); 
```

: 3. 그런데 `employeeBuddies`와 `managerBuddies`는 같은 객체를 가리키고 있다. 결과적으로 애초에 관리자만 담기로 했던 `managerBuddies`안에 일반 직원이 들어가 버리는 사고가 발생한다.

: 자바 컴파일러는 이러한 모순을 차단하기 위해 제네릭 타입간의 상속 변환을 금지한다.

배열은 제네릭과 다르게 동작한다.
: `Manager[]`는 `Employee[]`의 자식으로 인정된다.
```java
Manager[] managerBuddies = {ceo, cfo};
Employee[] employeeBuddies = managerBuddies; // 가능
```

: 그렇다면 배열은 위에서 발생한 사고가 발생하지 않을까?
```java
employeeBuddies[0] = lowlyEmployee; // 컴파일은 됨
```
- 배열은 이를 방지하기 위해 런타임에 `ArrayStoreException`이라는 예외를 던져서 방어한다. 즉, 배열은 실행중에 검사하고, 제네릭은 컴파일 중에 검사하여 아예 막는다는 차이가 있다.

제네릭 타입끼리는 상속 관계가 없지만, 제네릭 타입은 항상 원시 타입의 하위 타입으로 인정된다. 이는 레거시 코드와의 호환성을 위해서이다.
```java
Pair<Manager> managerBuddies = new Pair<>(ceo, cfo);
Pair rawBuddies = managerBuddies; // 가능, Pair는 원시 타입
```
하지만 원시 타입을 통해 데이터를 조작하면 안전성이 깨질 수 있다.
```java
rawBuddies.setFirst(new File("...")); // 경고만 뜨고 실행됨
```
물론 나중에 이 값을 꺼내서 `Manager` 변수에 넣으려고 할 때 `ClassCastException`이 발생하겠지만, 제네릭이 주는 컴파일 타임의 안정성은 잃게 된다.

마지막으로, 제네릭 클래스가 다른 제네릭 클래스나 인터페이스를 상속, 구현하는 것은 가능하다. 이는 타입 파라미터가 변하는 것이 아니라, 클래스 자체의 족보를 따르는 것이다.
- `ArrayList<Manager>`는 `List<Manager>`를 구현한다
- 따라서 `ArrayList<Manager>`를 `List<Manager>`변수에 대입할 수 있다.
하지만 여기서도 주의할 점은 `ArrayList<Manager>`는 `ArrayList<Employee>`나 `List<Employee>`와는 아무런 관계가 없다는 것이다. 


























