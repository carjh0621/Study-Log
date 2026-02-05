제네릭 메서드는 메서드 선언부에 자신만의 타입 파라미터를 가진 메서드이다. 중요한 점은 일반 클래스 내부에서도 제네릭 메서드를 정의할 수 있다는 것이다.

제네릭 메서드의 정의
ex) 일반 클래스 `ArrayAlg` 안에 정의된 제네릭 메서드 `getMiddle`
```java
class ArrayAlg
{
	public static <T> T getMiddle(T... a)
	{
		return a[a.length / 2];
	}
}
```
- 타입 파라미터 `<T>`는 접근 제어자(`public static`)와 반환타입 `T` 사이에 위치해야한다.

제네릭 메서드의 호출
: 호출 방식은 두가지가 있다.
- 1. 타입을 명시, 지정 : 메서드 이름 바로 뒤에 <>로 타입을 명시
	- `String middle = ArrayAlg.<String>getMiddle("John","Q.","Public");`
- 2. 타입 추론 : 대부분의 경우 타입을 생략해도 된다. 컴파일러가 전달된 인자의 타입을 보고 T를 추론한다.
	- `String middle = ArrayAlg.getMiddle("John", "Q.", "Public");`

타입 추론이 실패하는 경우
: 타입 추론은 대부분 잘 작동하지만, 서로 다른 타입의 인자를 섞어서 전달할 때 혼란이 발생할 수 있다.
ex)
``` java
double middle = ArrayAlg.getMiddle(3.14, 1729, 0);
```

- 1. 컴파일러는 인자들을 오토박싱(Autoboxing)한다
    - `3.14` -> `Double`
    - `1729`, `0` -> `Integer`
- 2. 컴파일러: `Double`과 `Integer`의 공통 조상(Supertype)을 찾아야 한다
    - 후보 1: `Number` 클래스
    - 후보 2: `Comparable` 인터페이스 (두 클래스 모두 이를 구현함)
- 3. 컴파일러 버전에 따라 다르지만, 두 가지 해석이 모두 유효하므로 모호하다는 난해한 에러 메시지를 낼 수 있다.
- 4. 해결책: 모든 파라미터를 `double`로 통일하여 명시. (`1729.0`, `0.0`)

