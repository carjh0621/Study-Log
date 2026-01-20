컴파일 시점이 아닌, 실행 시점에 동적으로 인터페이스를 구현하는 클래스를 만드는 기능

## When to use Proxies

컴파일 타임에는 어떤 인터페이스를 구현해야 할지 모르지만, 런타임에 특정 인터페이스들을 구현하는 객체를 만들어야 할 때가 있다.
그러나, 인터페이스는 스스로를 인스턴스화 할수 없다. (`new InterfaceName()` 불가하다. 또한 클래스를 만들려면 코드를 작성하고 컴파일해야하는데, 실행 중에 이를 수행하는 것은 어렵다.

해결책1. 실행 중에 문자열로 소스코드를 생성 -> 파일 저장 -> 컴파일러를 호출 -> 결과물(클래스 파일)을 다시 로드 
- 느리고, 프로그램 배포 시 컴파일러도 함께 배포해야함

해결책2. 런타임에 완전히 새로운 클래스(프록시 클래스)를 생성
- 빠르고 효율적이며, 지정한 인터페이스들을 완벽하게 구현한다

프록시 작동원리
- 프록시 클래스는 런타임에 생성되지만, 실제로 메서드가 호출되었을 때 무슨일을 할지에 대한 코드는 가지고 있지 않다. 대신, 모든 메서드 호출을 Invocation Handler(호출 처리기)로 넘긴다.
	- 프록시 객체는 껍데기 역할이다. 지정된 인터페이스의 메서드와 `Object` 클래스의 메서드를 가지고 있다. (`toString, equals`등)

`InvacationHandler`
- 모든 프록시 인스턴스는 `InvocationHandler` 인터페이스를 구현한 객체를 가지고 있어야 한다. 
- 이 인터페이스는 하나의 메서드를 가진다.
	- `Object invoke(Object proxy, Method method, Object[] args)`
	- `proxy`: 메서드가 호출된 프록시 객체 자신
	- `method`: 호출된 메서드에 해당하는 Method 객체.
	- `args`: 메서드에 전달된 인자 배열



## Creating Proxy Objects

프록시 객체는 `Proxy` 클래스의 정적 메서드인 `newProxyInstance`를 통해 생성된다. 이 메서드는 세 가지의 매개변수를 필요로 한다.
- 1. Class Loader
	- 프록시 클래스를 정의할 때 사용할 클래스 로더
- 2. Class Objects Array (인터페이스 배열)
	- 프록시 객체가 구현해야할 인터페이스들의 목록
- 3. Invocation Handler
	- 실제 메서드 호출을 가로채서 처리할 로직이 담긴 객체
```java
class TraceHandler implements InvocationHandler {
    private Object target; // 실제 객체 (예: Integer 500)

    public TraceHandler(Object t) {
        target = t;
    }

    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable {
        // 1. 암묵적 매개변수(target) 출력
        System.out.print(target);
        // 2. 메서드 이름 출력
        System.out.print("." + m.getName() + "(");
        // 3. 인자들 출력
        if (args != null) {
            for (int i = 0; i < args.length; i++) {
                System.out.print(args[i]);
                if (i < args.length - 1) System.out.print(", ");
            }
        }
        System.out.println(")");

        // 4. 실제 메서드 실행 (target 객체의 메서드 호출)
        return m.invoke(target, args);
    }
}
...
public class ProxyTest {
    public static void main(String[] args) {
        Object[] elements = new Object[1000];

        // 1~1000까지의 Integer를 프록시로 감싸서 배열에 저장
        for (int i = 0; i < elements.length; i++) {
            Integer value = i + 1;
            InvocationHandler handler = new TraceHandler(value);
            
            // 프록시 생성: Comparable 인터페이스를 구현하도록 함
            Object proxy = Proxy.newProxyInstance(
                ClassLoader.getSystemClassLoader(),
                new Class[] { Comparable.class }, 
                handler
            );
            elements[i] = proxy;
        }

        // 찾을 키 값 생성 (랜덤)
        Integer key = (int) (Math.random() * elements.length) + 1;

        // 이진 검색 수행
        // Arrays.binarySearch는 내부적으로 elements[i].compareTo(key)를 호출함
        int result = Arrays.binarySearch(elements, key);

        // 결과 출력
        if (result >= 0) System.out.println(elements[result]);
    }
}

```
- 자바가 런타임에 만든 프록시 클래스의 코드



프록시의 주요 사용 사례
- 원격 메서드 호출 라우팅: 메서드 호출을 네트워크 건너편의 원격 서버로 전달한다
- 이벤트 연결: 사용자 인터페이스 이벤트를 실행 중인 프로그램의 동작과 연결
- 디버깅용 추적: 메서드 호출 시 로그를 남겨 디버깅을 도움






















