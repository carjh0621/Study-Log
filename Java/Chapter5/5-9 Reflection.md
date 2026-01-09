
Reflection
- 프로그램이 실행 중에 스스로의 구조(클래스, 메서드, 필드 등)를 분석하고 조작하는 기술
- 용도:
	- 실행 중에 클래스 기능을 알아내야 할 때.
	- 어떤 클래스인지 몰라도 메서드를 호출하거나 객체를 만들 때.

일반 애플리케이션 개발자보다는 **도구 개발자**(UI 빌더, ORM 프레임워크 작성자 등)에게 주로 필요한 기능

## `Class` class

자바 런타임 시스템은 모든 객체에 대해 RTTI(Runtime Type Identification) 정보를 유지관리함.
이는 각 객체가 어떤 클래스에 속해 있는지 추적하며, JVM이 올바른 메서드를 선택하는데 사용됨.
이 정보를 담고 있는 특수한 클래스가 `java.lang.Class` 

`Class`  객체는 특정 클래스의 속성(이름, 필드, 생성자 등)을 설명한다. 
얻는 방법
- `Object.getClass()` : 이미 생성된 객체 인스턴스가 있을 때 사용
	- `Employee e = new Employee(); Class cl = e.getClass();`
- `Class.forName()` : 클래스 이름을 문자열로 알고 있을 때 사용
	- `String className = "java.util.Random"; Class cl = Class.forName(className);`
	- 예외처리 필요
- `T.class` : 타입을 알때사용
	- `Class cl1 = Random.class; Class cl2 = int.class; Class cl3 = Double[].class;`
	- primitive type에도 사용 가능

`getName()` : 클래스의 전체 이름 (패키지명 포함) 반환
- 일반 클래스: `java.util.Random`
- 배열:
	- `Double[].class.getName() -> [Ljava.long.Double;`
	- `int[].class.getName()->[I`

타입 비교.
- JVM은 각 타입에 대해 유일한 Class 객체를 관리한다. 따라서 `==` 로 Class 객체를 비교할 수 있다.
- `if (e.getClass() == Employee.class)`
	- 객체 `e`가 정확히 `Employee` 클래스의 인스턴스인지 확인. (서브클래스 x)
- `if (e instanceof Employee)`
	- 객체 `e`가 `Employee` 이거나 서브클래스인 경우 true

동적 인스턴스
- `Class` 객체를 이용해 런타임에 인스턴스 생성 가능하다.
- `getConstructor().newInstance()`
```java
Class cl = Class.forName("java.util.Random");
Object obj = cl.getConstructor().newInstance();
//기본 생성자를 찾아서 인스턴스 생성
```
- 기본 생성자가 없으면 예외 발생. (`InvocationTargetException`)


## Primer on Declaring Exceptions

자바는 오류 발생 시 Exception을 던짐. 
적절히 처리하지 않으면 stack trace(에러 메시지)를 출력.
예외 처리를 통해 프로그램 종료를 해결하거나 우회할 수 있다.


예외 종류
- Unchecked Exception
	- NullPointerException, ArrayIndexOutOfBoundsException
	- 컴파일러가 간섭하지 않는다. 
	- 프로그래머의 실수로 인해 발생
	- --> 원인을 제거해야함
- Checked Exception
	- ClassNotFoundException, IOException
	- 컴파일러가 강제한다.
	- 프로그래머의 노력과 무관하게 발생 가능 (`Class.forName("...") 등)
	- ---> try-catch, throws 등으로 처리를 하거나 선언해야함

`throws`
- 메서드 선언부에 `throws` 추가
- ex
```java
public static void doSomethingWithClass(String name)
	throws ReflectiveOperationException
{
	Class c1 = Class.forName(name); //예외 발생 가능
	...
}
```


## Resources

클래스 파일 (.class) 외에 프로그램 실행에 필요한 연고나 데이터 파일들을 Resource 라고 한다.
- 이미지, 사운드 파일, 메시지나 버튼 라벨이 적힌 텍스트 파일 등.
- 책의 제목이나 저작권 연도 같은 텍스트를 소스 코드 안에 하드코딩하는 것보다, 외부 텍스트 파일로 분리하면 수정과 관리가 쉬워짐.

리소스 찾기
- 프로그램을 배포할 때 보통 JAR 파일로 묶어서 배포. JAR 안의 파일은 일반적인 파일 시스템 경로로 접근할 수 없다.
- 자바는 `Class` 객체를 사용하여 해당 클래스가 있는 위치를 기준으로 리소스를 찾는다.
- 로딩 단계
	- `Class c1 = ResourceTest.class;` 리소스를 가지고 있는 클래스의 객체를 얻는다.
	- url이 필요한 경우
		- `URL url = c1.getResource("about.gif");`
	- 데이터를 직접 읽어야 하는 경우
		- `InputStream stream = c1.getResourceAsStream("data/about.txt");`

경로 지정 규칙
- 자바가 리소스를 검색하는 위치는 class path 내부이다.
- 상대경로
	- `c1,getResource("about.gif")`
	- `ResourceTest` 클래스가 속한 패키지 폴더 내부에서 찾는다.
	- 만약 `ResourceTest`가 `resources` 패키지에 있다면, `resources/about.gif`를 찾는다
- 절대경로
	- `c1.getResource("/corejava/title.txt")`
	- JAR 파일의 최상위 루트에서부터 찾습니다. 패키지 구조를 무시하고 루트 디렉토리의 `corejava` 폴더를 찾는다.



## Reflection to Analyze the Capabilities of classes

java.lang.reflect (리플렉션 라이브러리)
사용하면 실행 중에 클래스의 내부구조를 분석할 수 있다.

분석도구 (java.lang.reflect 패키지 내부 클래스들)
- `Field`: 클래스 내부의 멤버 변수 정보를 담고 있다. (타입, 이름 등)
- `Method` : 클래스 안의 메서드 정보를 담고 있다. (리턴 타입, 이름, 파라미터 타입 등)
- `Constructor` : 클래스의 생성자 정보를 담고 있다. (파라미터 타입 등 )
- 위 세개의 클래스는 공통적으로 `getName()` 메서드를 가직 있어서 항목의 이름을 확인할 수 있다.

Modifiers (제어자)
- 클래스나 