`printf` 처럼 인자의 개수가 정해지지 않은 메서드를 만들 때 사용.
이를 Varargs 라고 부른다.

Varargs
- 메서드를 호출할 때 파라미터 개수를 임의로 조절해서 넣을 수 있는 기능
- ex: `public void test(String... args)` (`...` 점 세개)

작동 원리
- 배열로 포장돼서 넘어간다.
- ex: `max(3.1, 40.4, -5)` --> 컴파일러가 `max(new double[] {3.1 , 40.4, -5})` 로 바꿈
- 메서드 내부에서는 배열처럼 다루면 된다.

`printf`
- `System.out.printf` --> `public PrintStream printf(String fmt, Object... args)`
- fmt는 고정, 뒤에 오는 인자들은 모두 `Object[] args` 배열로 들어온다.
- int 같은 primitive type --> autoboxing 돼서 `Integer` 객체로 변환된 뒤 배열에 들어간다.

주의점
- 무조건 마지막에 써야한다.
	- `(int... nums, String s)` x , `(String s, int... nums)` o
	- 배열을 직접 넣어도 된다.
	- main 에서
		- `public static void main(String... args)` 도 된다.

