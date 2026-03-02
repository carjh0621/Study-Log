**Object는 모든 클래스의 조상**
- 자바의 모든 클래스는 `Object` 클래스를 상속받는다.
- 다만, `public class [class name] extends Object` 라고 명시하지 않아도 된다. 자동으로 상속받는다
- 모든 클래스의 조상이므로 어떤 타입의 객체든 다 담을 수 있다. 
	- 다만, 그냥 담아두기만 하는 용도이므로, 형변환(캐스팅)을 통해 실제 그 객체의 기능을 사용할 수 있다.
		`Object obj = new Employee("Harry", 35000);`
		`Employee e = (Employee) obj;`

**primitive, array**
- 숫자, 문자, boolean, ... -> primitive 타입은 객체 x
- 배열은 객체
	- 객체 배열 뿐만 아니라 primitive 타입 객체(`int[]`)도 `Object`를 상속받는다.
	- `Object` 변수에 `int[]` 할당 가능

---
``` java
int[] numbers = new int[10];
Object arrayObj = numbers; //에러 x

System.out.println(arrayObj.toString());
//[I@14ae5a5
//자바에서 객체를 그냥 출력하면 // System.out.println(obj)
//자동으로 obj.toString()이 실행
//배열은 이 메서드를 오버라이드 하지 않아서, Object 클래스의 기본 동작을 따름.
//[ : 배열, I : int 타입, @ : 구분자, 14ae5a5 : 객체의 해시코드를 16진수로 바꾼 값.
// 객체의 메모리 주소가 아닌, 객체마다 고유한 ID(해시코드)
```

---

## equals Method

`Object` 클래스에 있는 `equals` 메서드는 두 객체의 주소값이 똑같은지 본다.

내용을 비교하고 싶으면 오버라이딩 해야 한다.
- `Employee` 객체 두개가 있는데, 이름/월급/입사일이 같으면 같은 사람
- `equals`를 오버라이딩 해서 내부 상태를 비교하게 만들어야 한다.

책에서 나온 `equals` 구현
1. 주소 비교: `if (this == otherObject) return true;`
	- 나 자신이랑 비교 --> 무조건 참
2. NULL 체크: `if (otherObject == null) return false;`
	- 비교 대상 x --> 거짓
3. 클래스 타입 비교: `if (getClass() != otherObject.getClass()) return false;`'
	- 서로 다른 클래스 --> 비교대상 x
4. 캐스팅: `Employee other = (Employee) otherObject;`
	- 클래스 같은 것 확인됨 --> 형변환
5. 필드 비교: `return name.equals(other.name) && ... ;`
	- 내부 상태 비교

`name.equals(other.name)` 
- `name`이 `null`이면 NullPointerException 
- `Objects.equals(a,b)`
	- 둘다 null 이면 true, 하나만 null 이면 false

**자식 클래스의 equals**
- 무조건 `super.equals(other)` 를 먼저 호출해서 부모 쪽 필드가 같은지 먼저 검사
- 통과하면 자식 클래스의 추가 필드를 비교



## equals and Inheritance

상속 관계에서의 `equals` 구현

`instanceof` vs `getClass`
- 부모 클래스와 자식 클래스가 있을 때 `equals` 처리에 문제 발생 가능
- `instanceof` 
	- `employee.equals(manager)` 는 true가 될 수 있다. (employee: 부모 manager : 자식)
	- `manager.equals(mangeer)` 는 false가 된다. (manager의 추가 필드까지 비교해야 해서)
	- 즉, 대칭성 규칙이 깨진다.
- `getClass()`
	- 클래스 타입이 정확히 같아야만 비교
	- 대칭성은 지켜짐

**`equals`가 지켜야 할 규칙**
1. **Reflexive:** `x.equals(x)`는 항상 true.
2. **Symmetric:** `x.equals(y)`가 true면 `y.equals(x)`도 true여야 함. (상속에서 주의해야 할 점)
3. **Transitive** `x=y, y=z`이면 `x=z`.
4. **Consistent:** 내용이 안 바뀌면 몇 번 호출해도 결과가 같아야 함.
5. **Null 아님:** `x.equals(null)`은 항상 false.

해결책
- 하위 클래스에서 비교 기준이 달라질 때 (필드 추가 등)
	- `getClass()` 사용
- 상위 클래스에서 비교 기준이 고정될 때
	- `instanceof` 사용
	- `equals` 를 `final`로 선언

**완벽한 `equals` 작성**
1. `this == otherObject` (주소 같으면 바로 true, 성능 최적화)
2. `otherObject == null` (null이면 false)
3. 클래스 비교:
    - 비교 기준이 달라질때: `getClass() != other.getClass()` return false
    - 비교 기준 고정: `!(other instanceof ClassName)` return false
4. 캐스팅: `ClassName other = (ClassName) otherObject;`
5. 필드 비교: 기본형은 `==`, 객체는 `Objects.equals`, 배열은 `Arrays.equals` 사용.

`@Override` --> 컴파일러 체크



## HashCode Method

hash code: 객체에서 파생된 정수값.  --> 해시 테이블에서 객체를 빨리 찾기 위해 사용된다. 
서로 다른 객체라면 해시코드도 달라야 좋음 (충돌 방지)

String vs StringBuilder
- String: 내용이 같으면 해시코드도 같음
- StringBuilder: 해시코드 메서드가 정의 x --> `Object` 기본 동작(주소 기반)을 따름.
	- 내용이 같아도 객체가 다르면 해시코드가 다름

규칙
- `x.equals(y)` 가 true면 , `x.hashCode() == y.hashCode()` 여야 한다.
- 만약 equals는 ID로 비교하게 해놓고, hashCode는 주소값 그대로 두면, HashMap에 넣은 데이터를 못찾는다.
- 해시코드가 같다고 꼭 equals가 true는 아니다.

해시코드 만드는 법
- 소수를 이용한 계산.
- 개선된 방식: `Objects.hashCode(필드)` , `Double.hashCode(값)` 등을 조합
- 최신: `Objects.hashCode(필드1, 필드2, ...)`

주의 
 - 배열 필드는 넣을 떄  `Arrays.hashCode(배열)` 써야 한다.
 - Record: 알아서 hashCode 변환
 - 가능하면 서로 다른 객체는 다른 해시코드를 가지게 해야함.

---
StringBuilder? 
- 가변(Mutable)
- 배열을 만들고 덧붙이거나 지움 
- String과 달리 새로운 객체를 만들어서 반환 x

해시테이블?
- 변수가 힙에 있는 객체를 찾아갈 때, 변수안에 들어있는 메모리 주소(reference)를 타고 바로 찾음.
- 해시코드는 개발자가 쓰는것. HashMap, HashSet 같은 자료구조를 쓸 때, 그 내부에서 객체를 분류하려고 미리 계산해둔 것. 
- JVM 내부적으로 관리하는 테이블(String Constant Pool 등)이 있긴 하지만, 일반 객체 접근은 주소값으로

equals를 부모 자식 모두 수정하면?
- 부모의 경우 책에서 처럼 수정하면 됨
- 자식의 경우 부모(`super`)의 해시코드에 자신의 것을 섞으면 된다.
	- `return Objects.hash(super.hashCode(), bonus);`

---


## toString Method

toString: 객체의 값을 문자열로 반환
Object의 기본 동작: 오버라이딩을 안 하면 클래스이름@해시코드 형태로 출력. --> 디버깅에 도움 x

호출
- 객체를 문자열과 + 할 때, 자동으로 호출
- `"Current: " + obj` 는 자동으로 `obj.toString()`을 호출
- `System.out.println(obj)` 내부적으로 toString() 호출

구현
- `getClass().getName()`을 사용하는 것이 좋음
	- 상속받은 클래스에서도 이 메서드를 재사용할 때, 올바른 클래스 이름이 출력되기 때문이다.
- 자식 클래스: `super.toString()` 을 호출하여 부모의 필드 정보를 먼저 찍고, `]` 뒤에 본인의 필드를 덧붙이는 방식을 사용

배열의 경우
- 배열도 객체. 다만, toString이 오버라이딩 x
- 따라서, 
	- 1차원 배열: `Arrays.toString(배열)`
	- 다차원 배열: `Arrays.deepToString(배열)`

디버깅하거나 로깅할때 중요

