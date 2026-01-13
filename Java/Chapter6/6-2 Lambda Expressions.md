
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

람다는 단 하나의 추상 메서드 (single abstract method, SAM)를 가진 인터페이스가 기대되는 자리에만 사용할 수 있다. 
이러한 인터페이스를 함수형 인터페이스라고 한다.























































