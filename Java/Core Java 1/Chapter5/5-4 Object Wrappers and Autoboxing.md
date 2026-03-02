
**Wrapper Class**
- primitive Type을 객체로 감싸는 클래스..?
- `int -> Integer` , `char -> Character` 
- `byte -> Byte`  , `double -> Double` 등
- 특징
	- **Immutable** (`String`이랑 비슷, 생성된 후에는 값을 절대 못 바꿈)
	- **Final** 상속 불가능
- 쓰는 이유
	- Generic 에는 primitive Type을 넣지 못한다.
	- `ArrayList<int>` (x)
	- `ArrayList<Integer>` (o)

**Autoboxing, Unboxing**
- autoboxing (primitive -> wrapper)
	- `list.add(3);` -> 실제동작: `list.add(Integer.valueOf(3));` 
	- 컴파일러가 자동 변환
- unboxing (wrapper -> primitive)
	- `int n = list.get(0);` -> 실제 동작: `int n = list.get(0).intValue();`
- `Integer n = 3; n++;` -> unboxing, 더하기, autoboxing 자동 수행

주의점 1 (객체 비교)
- 객체 비교에서 `==` --> 주소값 비교. 내용물 비교 x
- 다만, 자바는 메모리 절약을 위해서 -128 ~ 127 사이의 숫자는 캐싱된 포장된 객체를 재사용함
	- `Integer a =100; Integer b = 100; `, `a==b` --> true
	- `Integer a= 1000; Integer b = 1000;`, `a==b` --> false
- 따라서 `.equals()` 를 써야 함.

주의점 2 (nullpointerexception)
- `Integer`는 객체, 즉 `null`이 들어갈 수 있다.
- `Integer n = null'` 인 상태에서 `int x= n;` 이 실행되면, unboxing
- --> `n.intvalue()` 호출중에 **NullPointerException** 발생 . (컴파일러 에러 x)

주의점 3 (삼항 연산자 타입 승격)
- `Integer n = 1; Double x = 2.0;`
- `System.out.println(true ? n : x);`
- --> 결과는 `1.0` 
- 그 이유는 삼항연산자는 두 후보의 타입을 맞추려고 하기 때문에, `Integer` 를 `Double`로 변환해서 출력한다.

`parseInt` , `valueOf`
- `Integer.parseInt("123")` : 반환값이 `int` 
- `Integer.valueOf("123")` : 반환값이 `Integer` 
	- `new Integer(123)` -> deprecated (사용금지 권장), `valueOf` --> 캐싱 혜택


