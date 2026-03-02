
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



## Using Reflection to Analyze the Capabilities of classes

java.lang.reflect (리플렉션 라이브러리)
사용하면 실행 중에 클래스의 내부구조를 분석할 수 있다.

분석도구 (java.lang.reflect 패키지 내부 클래스들)
- `Field`: 클래스 내부의 멤버 변수 정보를 담고 있다. (타입, 이름 등)
- `Method` : 클래스 안의 메서드 정보를 담고 있다. (리턴 타입, 이름, 파라미터 타입 등)
- `Constructor` : 클래스의 생성자 정보를 담고 있다. (파라미터 타입 등 )
- 위 세개의 클래스는 공통적으로 `getName()` 메서드를 가직 있어서 항목의 이름을 확인할 수 있다.
- `getModifiers()`를 통해 `public`, `static` 같은 수식어 정보를 가져올 수 있다.


`Class` 클래스에는 이 정보들을 배열 형태로 가져오는 메서드들이 있음.
- public 만 가져오고, 상속 받은 것을 포함해서 가져옴
	- `getMethods()` `getFields()` `getConstructors`
- private, protected, public 모두 가져옴, 상속 받은 것 미포함
	- `getDeclareMethods()` `getDeclareFields()` `getDeclareConstructors()`


Modifiers (제어자)
- 클래스나 메서드 앞에 붙는 public, private, static, final 같은 키워드를 modifiers 라고 한다. 
- reflection은 이를 정수값으로 반환
- `java.lang.reflect.Modifier` 비트의 조합을 사람이 읽기 쉽게 바꿔줌
	- `Modifier.toString(int)` 
	- `Modifier.isPublic(int)` `ifFinal(int)`

ex)
```java
Scanner in = new Scanner(System.in);  
System.out.print("분석할 클래스 이름을 입력하세요 (예: java.util.ArrayList): ");  
name = in.next();

Class<?> cl = Class.forName(name);

Class<?> superCl = cl.getSuperclass();  
String modifiers = Modifier.toString(cl.getModifiers());
```




## Using Reflection to Analyze Objects at Runtime

reflection으로 객체 내부의 값을 보고 조작할 수 있음.

이전 섹션 - 클래스의 이름, 타입
이번 섹선 - 셀제 생성된 객체의 내용

필드의 값을 읽거나 쓰려면 `Field` 객체의 메서드를 사용
- `Object get(Object obj)`: `obj` 객체 안에 있는 해당 필드의 현재 값을 가져온다
- `void set(Object obj, Object newValue)`: `obj` 객체의 해당 필드를 `newValue`로 변경
```java
Employee harry = new Employee("Harry", 50000);
Class cl = harry.getClass();
Field f = cl.getDeclaredField("name"); // name 이라는 필드 정보를 가져옴
Object v = f.get(harry); // harry 객체의 name 값 반환
```


`setAccessible(true)`
- 위 예시 코드를 그대로 실행하면 에러가 발생.
- `name` 필드가 보통 private으로 선언되어있기 때문이다.
- 자바의 접근 제어자는 외부에서 private 에 접근하는 것을 막는다. (`IllegalAccessException`)
- `f.setAccessible(true)`: 해당 필드에 대한 자바의 접근 제어 검사를 끔. --> private 필드도 접근 가능
	- 이 메서드는 `Field, Method, Constructor` 의  공통 부모인 `AccessibleObject` 에 정의되어 있다.

ex)
```java
class SecretBox {  
    private String secret = "내 보물 1호";  
    private int code = 1234;  
  
    @Override  
    public String toString() {  
        return "SecretBox [secret=" + secret + ", code=" + code + "]";  
    }  
}
Class<?> cl = box.getClass();
Field secretField = cl.getDeclaredField("secret");
secretField.setAccessible(true);
System.out.println(secretField.get(box));
secretField.set(box, "쓰레기");
```



모듈 시스템의 제약
- 자바9 부터 도입된 모듈 시스템은 JDK 내부 클래스들에대한 캐슐화를 적용
- own 클래스들은 `setAccessible(true)`를 통해 열 수 있다. 하지만, `java.util.ArrayList` 같은 JDK 내부 클래스의 private 필드를 열려고 하면 자바가 막는다.
	- java 9~16 : 경고
	- java 17+ : `InaccessibleObjectException` 
- `--add-opens` 옵션을 줘야한다.
	- `java --add-opens java.base/java.util=ALL-UNNAMED MyProgram`

---
ex)
```java
$ java --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED ObjectAnalyzerTest.java

java.util.ArrayList[elementData=class java.lang.Object[]{java.lang.Integer[value=1][][],java.lang.Integer[value=4][][],java.lang.Integer[value=9][][],java.lang.Integer[value=16][][],java.lang.Integer[value=25][][],null,null,null,null,null},size=5][modCount=5][][]
```

`java.util.ArrayList`: 분석 대상 객체의 클래스 이름

`[elementData=class java.lang.Object[]{ ... }, size=5]`
- `ArrayList` 클래스에 정의된 자신만의 필드들
- `elementData`: `ArrayList`의 실체. 내부에 `Object[]` 배열을 숨기고 있는 것을 보여줌.
- **`size=5`:** 현재 데이터가 5개 들어있다
    
`java.lang.Integer[value=1][][]`
- 배열 안에 들어있는 첫 번째 데이터입니다.
- **`value=1`:** `Integer` 객체 내부의 실제 값(`int`).
- `[][]` (뒤쪽 괄호들): `Integer`의 부모 클래스인 `Number`, 그리고 그 부모인 `Object` 클래스에는 필드가 없어서 빈 괄호로 표시된 것

`null, null, null, null, null`
- `ArrayList`는 기본적으로 방을 10개(`DEFAULT_CAPACITY`) 만든다. 5개만 채웠으니 나머지 5칸은 비어있는(`null`) 상태로 배열이 유지되고 있음을 보여줌.

`[modCount=5]`
- `ArrayList`의 부모인 `AbstractList` 클래스의 필드입니다.
- `modCount`는 리스트가 몇 번 수정되었는지 기록하는 변수. (add를 5번 했으니 5)

`[][]` (맨 마지막 괄호들)
- 더 상위 부모인 `AbstractCollection`과 `Object` 클래스까지 올라가서 필드를 찾았으나, 아무것도 없어서 빈 괄호만 찍힌 것

---


## Using Reflection to Write Generic Array Code

배열이 꽉 찾을 때 크기를 늘려서 복사하고 싶을때, 모든 타입에 대해 동작하도록 만드는 방법. (제네릭하게)

`Object[]` 사용
- 단순하게 생각하면, 모든 클래스는 `Object` 를 상속받으니, `Object[]` 배열을 만들어서 복사하면 된다.
```java
// 나쁜 예시
public static Object[] badCopyOf(Object[] a, int newLength) {
    var newArray = new Object[newLength]; // 문제 발생 지점
    System.arraycopy(a, 0, newArray, 0, Math.min(a.length, newLength));
    return newArray;
}
```
- 문제점.
	- 이 메서드는 `Object[]` 타입의 객체를 생성한다.
	- `Object[]` 배열은 `String[]`이나 `Employee[]`로 형변환 할 수 없다.

reflection 사용
- 원래 배열과 똑같은 타입의 새 배열을 만들어야 한다.
- `java.lang.reflect.Array` 클래스의 기능을 사용
- 구현
	- `Object[]` 가 아닌 `Object` 로 받아야 한다. 
		- `int[]` 같은 원시 타입 배열은 `Object[]`로 형변환이 안 되지만, `Object`로는 처리 가능하기 때문
	- `a.getClass()`로 배열의 클래스 정보를 얻는다.
	- `cl.isArray()`로 진짜 배열인지 확인
	- `cl.getComponentType()`
		- `String[]`이면 `String.class`, `int[]`면 `int.class`를 반환
	- `Array.newInstance(componentType, newLength)`를 사용하여 정확한 타입의 새 배열을 생성
	- `System.arraycopy`를 사용해 내용을 복사


주요 메서드 (`java.lang.reflect.Array`)
- `static Object newInstance(Class<?> componentType, int length)`: 지정된 타입과 길이로 새 배열을 생성한다..
- `static int getLength(Object array)`: 배열의 길이를 반환(배열이 `Object` 타입으로 넘어왔을 때 유용)

ex)
```java
public static Object goodCopyOf(Object a, int newLength) {  
    Class<?> cl = a.getClass();  
    // 1. 배열인지 확인  
    if (!cl.isArray()) return null;  
    // 2. 배열의 구성 요소 타입(Component Type) 확인 (예: int, String)  
    Class<?> componentType = cl.getComponentType();  
    // 3. 현재 길이 확인 (리플렉션 사용)  
    int length = Array.getLength(a);  
    // 4. 해당 타입으로 새 배열 생성  
    Object newArray = Array.newInstance(componentType, newLength);  
    // 5. 배열 내용 복사  
    System.arraycopy(a, 0, newArray, 0, Math.min(length, newLength));  
    return newArray;  
}
```



## Invoking Arbitrary Methods and Constructors

자바에는 포인터가 없기 때문에 메서드의 주소를 다른 메서드에 전달하는 직접적인 방법이 없다.
하지만 reflection의 `Method` 클래스를 사용하면 실행 시간에 동적으로 메서드를 호출할 수 있다.

`Method.invoke()`
- `Method` 객체는 특정 클래스의 특정 메서드를 나타낸다. 이 객체의 `invoke` 메서드를 호출하면, 실제 메서드가 실행된다.
- `Object invoke(Object obj, Object... args)`
	- `obj`: 메서드를 호출할 대상 객체
		- 인스턴스 메서드를 호출하고싶다면, 실젝 객체를 넘겨야 한다.
		- 정적 메서드의 경우 대상 객체가 필요 없으므로, `null`을 넘겨야한다.
	- `args`: 실제 메서드에 전달할 매개변수들
	- 반환 타입이 `int` `double` 같은 원시 타입인 경우, 자동으로 `Integer` `Double`로 포장된다. 따라서 사용할 때 반드시 형변환 해야한다.


`getMethod`
- 특정 메서드를 실행하려면 먼저 그 메서드를 가리키는 `Method` 객체를 찾아야 한다.
```java
// 문법: getMethod("메서드이름", 매개변수타입1.class, 매개변수타입2.class, ...)
Method m1 = Employee.class.getMethod("getName");
Method m2 = Employee.class.getMethod("raiseSalary", double.class);
```

`Constructor.newInstance`
``` java
// 파라미터로 long을 받는 생성자 찾기
Constructor cons = Random.class.getConstructor(long.class);
// 생성자 실행 (인자 전달)
Object obj = cons.newInstance(42L);
``` 

reflection 호출의 단점
- 권장 x
- 오류 검증이 불가하다
- 성능이 저하된다
- 가독성이 저하된다






























