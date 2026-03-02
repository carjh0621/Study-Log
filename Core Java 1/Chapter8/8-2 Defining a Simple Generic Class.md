제네릭 클래스는 하나 이상의 타입 변수를 가진 클래스이다. 이 챕터에서는 데이터 저장의 복잡함을 배제하고 제네릭 메커니즘 자체에 집중하기 위해 간단한 `Pair` 클래스를 예로 들어서 살펴본다.

제네릭 클래스의 구조
: `Pair` 클래스는 두 개의 객체를 묶어서 저장하는 단순한 컨테이너이다.
- 정의: 클래스 이름 뒤에 <>로 묶인 타입변수 T를도입한다.  `public class Pair<T> {...}`
- 만약 첫번째 필드와 두번째 필드의 타입이 다르면, 여러개의 타입 변수를 정의할 수 있다. `public class Pair<T,U> { ... }`
- 타입 변수 T는 클래스 내부에서 필드 타입, 메서드반환 타입, 로컬 변수 타입 등으로 사용된다.
```java
private T first; // 필드 정의
public T getFirst() {return first;} //반환 타입
```

참고
: Naming Conventions
- `E` : 컬렉션의 요소 
- `K`,`V`: 테이블의 키,밸류
- `T` ,`U`, `S` : 일반적인 타입

제네릭 클래스의 인스턴스 화
: 제네릭 클래스를 사용할 때는 타입 변수 자리에 구체적인 타입을 대입한다. `Pair<String>`. 이때 제네릭 클래스는 일반 클래스를 찍어내는 공장처럼 동작한다고 생각하면 된다. `Pair<String>` 을 선언하면, T가 모두 `String`으로 바뀐 클래스가 있는 것처럼 동작한다.
- 교재 예시
```java
class Pair<T>  
{  
    private T first;  
    private T second;  
  
    public Pair() { first = null; second = null; }  
    public Pair(T first, T second) { this.first = first; this.second = second; }  
  
    public T getFirst() { return first; }  
    public T getSecond() { return second; }  
  
    public void setFirst(T newValue) { first = newValue; }  
    public void setSecond(T newValue) { second = newValue; }  
}
```