
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
// 1. 할당
// 2. 변환: 메서드를 객체로 변환
// 2. 인스턴스 생성: 힙에 `Consumer` 인터페이스를 구현한 이름없는 객체가 생성됨. `printer` 변수는 그 객체를 가리킴
 ...
 printer.accept("Hello");
 // printer가 객체이기 때문에 일반 객체처럼 메서드 호출 가능.
 ```

---



## Constructor References

생성자 참조
- 문법: `ClassName:::new`
- 동작: 람다식의 파라미터를 생성자의 인자로 전달하여 객체를 생성(`return new ...`)
- 대입되는 함수형 인터페이스의 파라미터 타입에 따라 호출되는 생성자가 결정된다.

ex) `Person` 클래스에 생성자가 2개 있을 때
```java
Class Person{
	Person() { ...}
	Person(String name) {...}
}
```
- `Supplier` 
	- `Supplier<Person> s = Person::new;` -> `new Person()` 호출
- `Function`
	- `Function<String, Person> f = Person::new;` -> `new Person(String)` 호출


배열 생성자 참조
- 문법: `TypeName[]::new`
- 대응되는 람다식: `n -> new TypeName[n]`
- 파라미터: 배열의 길이를 받는다.
- ex)
```java
IntFunction<int[]> arrayMarker = int[]::new;

int[] myArr = arrayMaker.apply(5); 
```
- 필요한 이유
	- java에서는 `new T[n]` 처럼 제네릭 타입으로 배열을 바로 만드는 것이 금지됨.
	- 보통 라이브러리(Stream API 등)는 배열을 반환할 때, `Object[]`를 반환한다.
	- `Object[] people = stream.toArray()`;  형변환을 해도 에러발생 가능
	- 그렇기에 `Person[] people = stream.toArray(Person[]::new);`
		- stream 이 데이터 개수를 세고 
		- 넘겨받은  `Person[]::new` 에 개수를 넣고 실행 -> `new Person[size]` 생성
		- 생성된 배열에 데이터를 채워서 반환 

---
`Supplier<Person> s = Person::new` 
- Anonymous Object 를 힙에 생성, s 는 그 객체를 가리킨다. 
- Supplier의 추상 메서드는 `T get()`.  `s.get()` 호출하면 내부적으로 `new Persin()` 호출

`java.util.function` 자세히

| 인터페이스 이름         | 추상 메서드              | 역할                    |
| ---------------- | ------------------- | --------------------- |
| `Supplier<T>`    | `T get()`           | 아무것도 안 받고 주기만 함 (공급자) |
| `Consumer<T>`    | `void accept(T t)`  | 받아서 쓰고 버림 (소비자)       |
| `Function<T, R>` | `R apply(T t)`      | T를 받아서 R로 바꿈 (변환기)    |
| `Predicate<T>`   | `boolean test(T t)` | T를 받아서 참/거짓 판별        |
파라미터 개수가 다른 변형
- `Bi...` 시리즈: 인자를 2개 받는 버전
    - `BiConsumer<T, U>`: `void accept(T, U)`    
    - `BiFunction<T, U, R>`: `R apply(T, U)`
    - `BiPredicate<T, U>`: `boolean test(T, U)`
- `UnaryOperator<T>`: 입력과 출력 타입이 같은 `Function` (예: `T` -> `T`)
- `BinaryOperator<T>`: 입력 2개와 출력 타입이 모두 같은 `BiFunction` (예: `T, T` -> `T`)

기본형(Primitive) 특화 변형 (성능 최적화용)
- 제네릭 `<T>`는 `Integer` 같은 객체만 들어갈 수 있어서, `int` 같은 기본형을 쓰려면 오토박싱(Auto-boxing) 비용이 든다. 이를 막기 위해 만들어짐.
- `IntFunction<R>`: `int`를 받아서 `R`을 리턴 (`apply(int value)`)
- `IntConsumer`: `int`를 받아서 소비 (`accept(int value)`)
- `IntPredicate`: `int`를 검사 (`test(int value)`)
- (`Long...`, `Double...` 시리즈도 동일하게 존재)

제너릭으로 배열 생성이 불가하다의 의미
```java
class MyBox<T> {
    public void makeArray() {
        // 에러! 컴파일러는 T가 뭔지 몰라서 배열을 만들 수 없음
        T[] arr = new T[10]; 
    }
}
```
- **가능:** `Integer[] arr = new Integer[10];` (구체적인 타입 명시)

Stream?
- 데이터의 흐름, 공장 컨테이어 벨트
- 예시: `names`라는 이름 리스트에서, "Kim"으로 시작하는 이름만 골라서 대문자로 바꾼 뒤 정렬하고 싶다.
``` java
// 예전 방식
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("Kim")) { // 1. 필터링
        String upperName = name.toUpperCase(); // 2. 변환
        result.add(upperName); // 3. 담기
    }
}
Collections.sort(result); // 4. 정렬
```
```java
//Stream 방식
List<String> result = names.stream()  // 1. 컨베이어 벨트에 올리기
    .filter(name -> name.startsWith("Kim")) // 2. 작업자1: 김씨만 통과시켜 
    .map(name -> name.toUpperCase())        // 3. 작업자2: 대문자로 바꿔 (Function)
    .sorted()                               // 4. 작업자3: 순서대로 줄 세워
    .collect(Collectors.toList());          // 5. 결과: 리스트로 포장해 (Terminal)
```
- 컨베이어 벨트(Stream)는 껍데기일 뿐이고, 거기서 실제로 무슨 일을 할지(거르는 기준, 바꾸는 방법)는 우리가 알려줘야한다
- 그 일하는 방법을 가장 간단하게 적어서 던져주는 게 바로 람다식(또는 메서드 참조)
	- `stream.map(Person::new)` -> 지나가는 재료(String)를 가지고 `Person` 객체로(new) 바꿔라
	- 교재 예제를 다시 보면
``` java
// names: ["철수", "영희", "민수"]
Person[] people = names.stream()
    .map(Person::new)             // 이름(String)이 지나가면 Person 객체로 바꿔라
    .toArray(Person[]::new);      // 다 끝난 Person들을 Person 배열에 담아라
```
- `names.stream()`: 이름 데이터를 벨트에 올림.
- `.map(Person::new)`: 지나가는 이름 하나하나를 붙잡고 `new Person(이름)`을 실행해서 사람 객체로 변신시킴.
- `.toArray(...)`: 이제 벨트 끝에 쌓인 사람 객체들을 배열로 만들어서 내놔.
    - (이때 `Person[]::new`를 줘서 "Person 배열 만드는 법"을 알려줌)
---


## Variable Scope


```java
public static void repeatMessage(String text, int delay) {
    ActionListener listener = event -> {
        // text는 이 람다가 실행될 때쯤이면 이미 메모리에서 사라졌을 텐데?
        System.out.println(text); 
    };
    new Timer(delay, listener).start();
}
```
- `repeatMessage` 가 호출되면 text, delay  두 지역 변수가 스택에 생긴다. 그리고 메서드가 종료되면 이 스택 프레임은 사라지고 변수들도 함께 소멸해야 정상이다.
- 즉 타이머가 시작되고, repeatMessage 메서드는 리턴된다. --> 스택 프레임 파괴 = 지역변수 소멸
- 그러면 Timer가 ActionListener의 actionPerformed 메서드를 호출한다. 그러면 text 변수가 필요한데 스택이 사라져서 문제가 생긴다.
- 하지만, 위 코드는 정상 작동한다. 그 이유는 람다 표현식은 Closure 이기 때문이다.
	- 람다의 3요소
		- 코드 블록, 파라미터, 자유 변수의 값(Values of free variables)
	- 자유변수 : 파라미도 아니고, 람다 내부에서 정의되지도 않은, 바깥쪽 변수를 의미
	- 캡처(capture): 람다 표현식이 생성되는 순간, 참조하는 바깥 변수의 값을 복사해서 자기 내부에 저장해둔다. --> 원본 `text` 가 사라져도, 람다는 복사본을 가지고 있으므로 나중에 실행될 때 문제가 없다.

제약 조건 (Effectively Final)
- java는 람다 내부에서 바깥 변수를 사용할 때 제약을 둔다.  = 읽기만 하고 변경은 하지 마라
- 람다 안에서 캡처한 변수의 값을 바꾸려고 하면 컴파일 에러가 난다
```java
int start = 10;
ActionListener listener = event -> {
	start--; // 에러 
}
```
- 람다 밖에서 값이 바뀌는 변수도 람다 안에서 쓸 수 없다.
```java
for(int i=0; i<10;i++){
	ActionListener listener = event -> System.out.println(i); // 에러
}
```
- 이유: 동시성 문제 (Concurrency Safety)
	- 여러 쓰레드가 동시에 하나의 변수를 줄이려고 하면, 데이터가 꼬이는 race condition이 발생
	- java는 이를 막기 위해서 값이 변하지 않는 변수 (effectively final)만 캡처할 수 있는 규칙을 정함. (final 키워드를 붙이지 않았더라도, 초기화된 이후에 값이 한 번도 바뀌지 않는 변수)

스코프 규칙 (Naming & Shadowing)
- 람다 표현식의 body(`{...}`)는 독립적인 새로운 세상이 아니라, 감싸고 있는 메서드와 같은 스코프로 취급된다
- 이름 충돌(Name Conflicts)
	- 같은 스코프이므로, 바깥에 있는 변수 이름과 똑같은 이름의 파라미터나 지역 변수를 만들 수 없다.
```java
Path first = Path.of("/usr/bin");
Comparator<String> comp = (first, second) -> ...
// 람다의 스코프 바깥에 first가 있는데 람다 내부에 first 라고 이름을 붙이면 안된다.
```
- `this` 키워드
	- Anonymous Class에서 `this` 는 자기 자신을 가리킨다
	- 람다에서의 `this`는 람다를 감싸고 있는 클래스의 인스턴스를 가리킨다
```java
public class Application{
	public void init() {
		ActionListener listener = event -> {
			System.out.println(this.toString());
			//this -> Application 객체
		};
	}
}
```



## Processing Lambda Expressions

람다를 받는 메서드를 만드는 이유 : 지연 실행 (Deferred Execution)
- 지금 당장 실행하는게 아니라, 나중에 필요할 때 실행하기 위해서.
- 코드를 나중에 실행해야 하는 상황들
	- 별도의 쓰레드, 반복 실행, 이벤트 처리, 조건부 실행(Lazy Evaluation)

교재의 예제 : `repeat` 메서드
- 상황에 따라 어떤 함수형 인터페이스를 파라미터로 선택해야하는지
- 1. 단순 반복 (`Runnable`)
	- 단순히 "Hello"를 10번 출력. 인자, 리턴값 x 
	- --> 적합한 표준 인터페이스 : `Runnable`(`void run()`)
```java
public static void repeat (int n, Runnable action){
	for(int i=0 ; i<n ; i++){
		action.run(); // 여기서 람다식의 본문이 실행됨
	}
}
...
repeat(10, () -> System.out.println("Hello"));
```
- 2. 인자가 필요한 반복 (`IntConsumer`)
	- 현재 몇 번째 반복인지를 출력하고 싶을때. 그러면 람다식이 정수를 받아야 한다. 이때는 `Runnable` 이 아니라 `IntConsumer`(`Void accept(int)`) 이 적합하다
```java
// IntConsumer: int 값을 하나 받아서 소비하는 인터페이스
public static void repeat(int n, IntConsumer action){
	for(int i=0; i<n; i++){
		action.accept(i); // 루프의 변수 i를 람다식의 인자로 전달함 
	}
}
...
repeat(10, i -> System.out.println("Countdown: " + (9-i)));
//i는 repeat 메서드 안에서 넘겨준 값
```


`java.util.function` 패키지 정리... (앞에서 두번 정리함)


`@FunctionalInterface` 
- 직접 함수형 인터페이스를 만들 때, 이 어노테이션을 붙여주는 것이 좋다. (필수 x)
```java
@FunctionalInterface
public interface MyChecker {
	boolean check(int n);
}
```
- 장점1: 추상 메서드를 실수로 2개 만들면 컴파일러가 에러를 띄워준다.
- 장점2: Javadoc에 함수형 인터페이스임이 명시되어, 다른 개발자가 람다식으로 써도 된다는 것을 알 수 있다.


Method Chaining, `transform`
- 코드를 왼쪽에서 오른쪽으로 물 흐르듯 읽을 수 있을 때 = method chaining. 하지만 타입이 바뀌는 순간 보통 체인이 끊긴다.
```java
String input = "12345";

// String 메서드들은 체이닝이 됨
String cleaned = input.strip(); 

// 여기서 체인이 끊김. 
// String을 BigInteger로 바꾸려면 생성자에 넣어야 함 (감싸는 형태)
BigInteger bigNum = new BigInteger(cleaned);

// 다시 BigInteger 메서드 시작
boolean result = bigNum.isProbablePrime(20);
```
- 위의 과정을 한 줄로 쓰면 다음과 같다
- `boolean result = new BigInteger(input.strip()).isProbablePrime(20);`
- `transform` 은 본인(객체)을 함수에 넣어서 결과물로 바꾸는 메서드. 이걸 쓰면 체이닝을 계속할 수 있따.
```java
boolean result = input.strip()                    // 1. 공백 제거 (String)
                      .transform(BigInteger::new) // 2. (String -> BigInteger)
                      .isProbablePrime(20);       // 3. 소수 판별 (BigInteger)
```

Java의 보수성: 왜 모든 객체(`Object`)에 `transform`을 안 넣어줬나?
- Java 8 때 `Function` 인터페이스가 생겼을 때 넣었어야 했는데 타이밍을 놓쳤다.
- 뒤늦게 `Object` 클래스에 메서드를 추가하면, 전 세계 어딘가에 이미 `transform`이라는 메서드를 쓰고 있던 다른 사람들의 코드와 충돌이 날 수 있다.
- 그래서 Java는 불편함을 감수하고서라도 기존 코드와의 호환성(Commitment)을 지키는 선택을 함. (`String` 클래스에만 `transform`을 넣어줌)





















