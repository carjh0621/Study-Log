
나중에 실행될 코드 블록을 어딘가로 전달하고 싶을때, java는 완전한 객체 지향 언어이기 때문에, 코드 그 자체만 전달할 수 없다. 코드를 전달하려면 , 그 코드를 담고 있는 객체를 생성해서 전달해야 했다. 
- Timer의 경우 
	- Worker 클래스로 ActionListener 인터페이스를 구현하고 actionPerformed 메서드 안에 작동할 코드블록을 작성한다. 그리고 이 클래스의 인스턴스를 Timer에 전달한다.
	- 즉, 코드블록을 전달하기 위해서 너무 많은 부분들이 따라오게 된다.


## Syntax of Lambda Expressions

기본 구조
- (파라미터들) -> 표현식
- ex) 두 문자열의 길이를 비교하는 코드
	- `(String first, String second) -> first.length() - second.length()`
		- `(String first, String second)`: 메서드의 파라미터 정의
		- `->`: 파라미터와 바디(Body)를 구분하는 연산자
		- `first.length() - second.length()`: 실행될 코드이자 리턴값. `return` 키워드가 없도 결과가 반환

문법 변형
- 코드 블록이 여러 줄일 때. 중괄호 사용. 이때는 return 명시해야함
```java
(String first, String second) -> {
	if (first.length() < second.length()) return -1;
	else if (first.length() > second.length()) return 1;
	else return 0;
}
```
- 파라미터가 없을 때. 빈 괄호 사용
```java
() -> {for (int i=100; i>=0; i--) System.out.println(i); }
```
- 컴파일러가 문맥상 파라미터의 타입을 추론할 수 있다면, 타입을 생략할 수 있다.
```java
Comparator<String> comp = (first, second) -> first.length() - second.length();
// Comparator<string> 인터페이스에 대입되므로 String임이 추론된다
```
- 파라미터가 하나일 때, 괄호조차 생략 가능하다
```java
ActionListener listener = event -> System.out.println("The time is "+ Instant.ofEpochMilli(event.getWhen()));
//(event) 대신 event 사용가능
```
- var 키워드 사용. 타입 추론을 사용하면서도 어노테이션을 부텨야 할 때 사용.
```java
(@NonNull var first, @NonNull var second) -> first.length() - second.length()
```

주요 규칙
- 람다식의 리턴 타입은 명시x. 문맥에 따라 자동으로 결정.
	- `(String first, String second) -> first.length() - second.length()` 는 int를 반환하는 문맥에서 사용 가능
- 모든분기에서 값을 반환하거나 모두 반환하지 않아야 한다.
	- `(int x) -> { if (x>=0) return 1;}`
	- x<0 일때 리턴값이 정의되지 않았으므로 컴파일 에러


## Functional Interfaces

함수형 인터페이스
- 추상 메서드가 단 하나만 존재하는 인터페이스를 의미
- 구현해야 할 메서드가 하나뿐이므로, 람다식이 그 메서드의 구현체임이 명확해진다.
- 다만, `default`메서드, `static` 메서드, `Object` 클래스의 메서드(`toString`, `clone`등)를 재선언한 것은 추상 메서드 개수에 포함되지 않는다.
- ex) Comparator
	- Comparator 인터페이스는 여러 메서드를 가지고 있지만, 추상 메서드는 `compare` 하나뿐이므로 함수형 인터페이스이다.
	- `Arrays.sort(words, (first,second) -> first.length() - second.length());`


다른 언어의 방식 (Function Types)
- JavaScript, Python, C#, Scala 등의 언어들은 함수 타입을 지원한다.
- 함수의 이름이나 소속된 인터페이스 이름은 중요하지 x.  파라미터가 무엇이고 리턴타입이 무엇인가 라는 signature만 맞으면 된다.
- JavaScript: `var add = (x,y) -> x + y;`
Java의 방식 (Nominal Typing)
- 자바의 경우는 타입의 이름이 매우 중요하다. 자바 설계자들은 람다를 위해 새로운 함수 타입을 언어에 추가하는 대신, 기존의 인터페이스 시스템을 활용하기로 했다.
- ex) `Arrays.sort` 는 `Comparator` 인터페이스를 받도록 되어있다.
	- 만약 함수 타입을 새로 만들었으면, 
	- 기존 `Arrays.sort` 는 `Comparaotr` 객체를 원하지만, 새로 만든 람다 `(a,b) -> a-b`는 `Function Type` 이다.
	- 결국 타입 불일치라 기존 `Arrays.sort`에 람다식을 넣을 수 없게 된다.
	- 결국 자바 설계팀은 람다를 지원하기 위해 `Arrays.sort`를 포함한 수천개의 API를 오버로딩 해서 다시 만들어야 했을 것이다.
- 그렇기에 java의 경우 람다를 Comparator 인터페이스인 척 하게 만들었다.

Object 에는 대입이 안된다.
```java
// 컴파일 에러 발생!
Object obj = (x, y) -> x + y; 
```
- 컴파일러는 "목적지(Target Type)"를 보고 람다를 해석한다. Java 컴파일러가 람다식을 해석하는 순서는 다음과 같다.
	1. 대입되는 곳의 타입(Target Type)을 본다.
	2. 그 타입이 함수형 인터페이스인지 확인한다.
	3. 그 인터페이스 안에 있는 단 하나의 추상 메서드(Method Signature)를 찾는다.
	4. 람다식의 파라미터와 리턴값이 그 메서드와 일치하는지 확인한다.
- Object에 대입할 때 일어나는 일
	1. 대입되는 곳의 타입은 `Object`
	2. `Object` 클래스를 본다.
	3. 문제 발생: `Object`에는 `run()`, `execute()`, `compare()` 같은 추상 메서드가 없다. (Object는 클래스이지 함수형 인터페이스가 아님)
	4. 컴파일러 입장: 이 람다식 `x + y`를 `Object`의 어떤 메서드인 척하고 포장해야 하는지 모름 -> 에러
- 해결 방법
	- 컴파일러에게 이건 `Comparator`라고 힌트(캐스팅)를 주거나, 명확한 인터페이스 타입 변수에 담아야 한다.
```java
// 가능: 컴파일러가 Comparator의 compare 메서드와 매칭시킴
Comparator<Integer> comp = (x, y) -> x + y; 

// 가능: Object에 담고 싶다면 먼저 형변환을 통해 정체를 밝혀야 함
Object obj = (Comparator<Integer>) (x, y) -> x + y;
```


`java.util.function`패키지 표준 인터페이스
- Java는 자주 사용되는 패턴을 위해 미리 정의된 함수형 인터페이스들을 `java.util.function` 패키지에 제공한다.
- BiFunction `<T,U,R>`
	- 파라미터 두 개(T,R)를 받아서 결과 (R)를 리턴한다.
	- 람다식을 `BiFunction` 변수에 저장할 수는 있지만, `Arrays.sort` 같은 기존 API는 `BiFunction`이 아니라 `Comparator`라는 명확한 목적을 가진 인터페이스를 원하기 때문에 바로 사용할 수 없는 경우가 많다...
- Predicate `<T>`
	- 파라미터 하나 (T)를 받아 boolean을 리턴. (`boolean test(T t)`)
	- 조건을 검사하여 필터링할 때 주로 사용
	- `list.removeIf(e -> e == null));`
- Supplier `<T>`
	- 파라미터 없이 값(T)을 리턴
	- 지연 연산에 유용
	- ex) Objects.requireNonNullElseGet 
		- `LocalDate hireDay = Objects.requireNonNullElse(day, new LocalDate.of(1970, 1, 1));`
		- `LocalDate hireDay = Objects.requireNonNullElseGet(day, () -> new LocalDate.of(1970, 1, 1));`
		- 전자의 경우 day가 null이 아니라면 `LocalDate` 객체를 생성했다가 버리는 낭비. 
		- 후자의 경우 `() -> new ...` 부분은 day가 null인 경우에만 실행. 이는 불필요한 객체 생성을 막아 성능을 최적화 할 수 있다.



## Method Reference

람다 표현식이 단 하나의 기존 메서드만을 호출할 때, 이를 더 간결하게 표현하는 문법. (굳이 람다식을 쓸 필요가 없고, 이미 있는 메서드를 바로 지목)
- `::`을 사용.
- 람다식과 마찬가지로 함수형 인터페이스의 인스턴스를 생성. (객체가 아님)
	- 객체가 아님? 
		- 객체는 필드와 메서드를 묶은 것. 메서드 참조는 필드가 없고 실행하는 기능만 있음. 전통적인 객체와 다름.
		- new 를 쓰면 객체가 힙 메모리에 생김. 다만 메서드 참조는 의향일 뿐, 이것이 어떤 인터페이스 변수에 대입되는 순간에 그 인터페이스의 객체로 변환
		- 컴파일러 관점에서는 인터페이스 변수에 넣을때 알아서 해당 인터페이스를 구현한 기능만 있는 익명 클래스의 인스턴스와 유사한 것을 동적으로 생성함
- ex)
	- 람다식: `event -> System.out.println(event)` :이벤트가 들어오면, 코드블록 실행
	- 메서드 참조: `System.out::println` : 이벤트가 들어오면, System.out 객체의 println 메서드에 넘김



패턴
- `object::instanceMethod` 
	-  이미 존재하는 인스턴스의 메서드를 호출할 때 사용. 람다의 파라미터가 해당 메서드의 인자로 전달됨.
	- ex) `System.out::println` 
		- 람다식: `x->System.out.println(x)`
		- `System.out` 이라는 객체는 이미 힙 메모리에 존재. 람다로 들어온 x를 그 객체의 `println` 메서드에 전달함.
- `Class::staticMethod`
	- 정적 메서드를 호출할 때 사용한다. 람다의 모든 파라미터가 정적 메서드의 인자로 그대로 전달
	- ex) `Math::pow`
		- 람다식: `(x,y) -> Math.pow(x,y)`
- `Class::instanceMethod`
	- 클래스 이름이 앞에 오지만, 정적 메서드가 아닌 인스턴스 메서드를 호출할때. 람다의 첫 번째 파라미터가 메서드의 주체가 되고 나머지가 파라미터가 된다.
	- ex) `String::compareToIgnoreCase`
		- 람다식: `(x,y) -> x.compareToIgnoreCase(y)`

주의점
- `this`, `super` 사용가능
	- `this::equals` -> `x -> this.equals(x)`
	- `super::greet` -> `() -> super.greet()`
- Null safety의 차이
	- 일반 람다식과 메서드 참조는 `NullPointerExceptinon` 발생 시점이 다를 수 있다.
	- `separator::equals`, separator가 null인 경우에 
	- 메서드 참조의 경우 정의하는 순간에 예외 발생 (evaluating time)
	- 람다식의 경우는 실제 호출될 때 예외 발생. (invocation time)
- 오버로딩 해결
	- System.out.println 처럼 이름이 같은 메서드가 여러 개일 경우는?
	- 컴파일러가 문맥을 보고 대입하려는 함수형 인터페이스의 파라미터 타입과 가장 잘 맞는 메서드를 자동으로 선택. 
- 사용 불가능한 경우
	- 람다식 내부에서 메서드 호출 외에 다른 로직이 조금이라도 섞이면 메서드 참조로 바꿀 수 없다.
	- `s -> s.length() ==0` 
		- 메서드 호출 외에 비교 연산이 있으므로 변환이 불가하다. 

메서드 참조는 람다식의 파라미터를 지정한 메서드의 파라미터로 매핑해준다.

---
 **함수형 인터페이스 변수에 람다식 또는 메서드 참조를 할당하면, 컴파일러가 알아서 해당 인터페이스를 구현한 인스턴스(객체)를 생성해서 넣어준다**
 ```java
 Consumer<String> printer = System.out::println;
 ...
 printer.accept("Hello");
 ```
1. 할당
2. 변환: 메서드를 객체로 변환
3. 인스턴스 생성: 힙에 `Consumer` 인터페이스를 구현한 이름없는 객체가 생성됨. `printer` 변수는 그 객체를 가리킴
---




































