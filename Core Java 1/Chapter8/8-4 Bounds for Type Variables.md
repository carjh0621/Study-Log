제네릭 메서드나 클래스를 만들다 보면 T에 아무 타입이나 들어오는게 아니라 특정 기능을 가진 타입만 들어오도록 제한해야할 때가 있다.

예를들어, 배열에서 최솟값을 찾는 `min`메서드를 만든다고 가정해 보자.
```java
public static <T> T min(T[] a){
	// ...
	if (smallest.compareTo(a[i]) > 0) 
	// ...
}
```
여기서 `smallest` 는 T 타입이다. 그런데 T가 임의의 클래스라면, 그 클래스에 `compareTo` 메서드가 있다는 것을 어떻게 보장할 필요가 있다. 
이 문제를 해결하기 위해 T가 반드시 `Compareable` 인터페이스를 구현한 클래스여야 한다고 명시할 수 있다.
```java
public static <T extends Comparable> T min(T[] a) ...
```
- 이제 이 메서드는 `String`, `LocalDate` 처럼 비교 가능한 객체들의 배열만 받을 수 있다. 

문법적 특징
: `implements` 가 아닌 `extends`?
- `Comparable`은 인터페이스지만 `<T implements Comparable>` 이라고 쓰지 않고 `extends`를 쓴다. 제네릭에서 `extends` 는 상속이라는 의미보다는 ~의 하위타입 이라는 더 포괄적인 의미로 쓰인다.
- Java 설계자들은 `sub`같은 새 키워드를 추가하는 대신 `extends` 를 활용하기로 결정
: 다중 제한.
- 타입 변수에 여러 개의 제한을 둘 수도 있다. 이때는 `,`가 아닌 `&` 로 구분한다.
- `T extends Comparable & Serializable`
- 제한 목록 중에 클래스가 있다면 반드시 가장 먼저 와야 한다. 