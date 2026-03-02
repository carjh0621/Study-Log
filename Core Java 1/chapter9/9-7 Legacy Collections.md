자바 컬렉션 프레임워크가 등장하기 전 자바 1.0 시절부터 존재했던 컬렉션들.
자바 1.0에서의 데이터 관리 클래스는 vector, hashtable, enumeration 등 몇개 없었다. 자바 1.2에서 컬렉션이 나오면서 호환성이 문제가 되었다.
그래서 구형 클래스들에 새로운 인터페이스를 구현하도록 하여 컬렉션 프레임워크 족보에 포함시켰다.
## The Hashtable Class

`java.util.Hashtable`은 키-값 쌍으로 데이터를 저장하는 자료구조. `HashMap`과 목적 및 인터페이스가 거의 완벽하게 동일하다.
허나 모든 주요메서드에 `synchronized` 처리가 되어 있다. -> 단일 스레스에서의 락 오버헤드로 인한 성능저하
그렇기에 단일 스레드 환경에서는 `HashMap`을 쓰고, 또한 멀티 스레드 환경에서는 `ConcurrentHashMap`을 쓴다. (훨씬 정교하고 빠름)

## Enumerations

`java.util.Enumeration`은 컬렉션 내부의 요소들을 순회하기 만들어진 인터페이스. `Iterator`와 역할이 같음


## Property Maps

`java.util.Properties` - 프로그램의 설정 옵션을 저장하기 위해 만들어진 특수한 형태의 맵 자료구조.
- 일반 맵과 달리 키-값이 모두 무조건 문자열
- 맵에 들어있는 데이터를 텍스트 파일로 쉽게 저장하고, 파일에서 쉽게 읽어올 수 잇다..
- 기본값을 제공하는 보조 테이블 기능이 있음

(메모리에서)데이터 넣기: `settings.setProperty("width","600.0");`
(메모리에서)데이터 빼기: `settings.getProperty("width");`
이 데이터를 파일(`.properties`)로 입출력할 때 주의사항이 있다.
파일을 다룰 때 과거 방식인 `InputStream/OutputStream`을 사용하면, 자바는 ISO 8859-1인코딩을 기본으로 사용한다. 이 경우 한글이나 최신 유니코드 문자가 깨니거나, 이스케이프 문자로 변환된다.
따라서 반드시 `Reader/Writer` 객체와 `StandardCharsets.UTF_8`을 조합하여 명시적으로 UTF-8 인코딩을 지정해야 안전하게 저장하고 불러올 수 있다.
```java
// 파일로 저장할 때 (올바른 방식)
var out = new FileWriter("program.properties", StandardCharsets.UTF_8);
settings.store(out, "Program Properties"); // 두 번째 인자는 파일 상단에 적힐 주석

// 파일에서 읽어올 때 (올바른 방식)
var in = new FileReader("program.properties", StandardCharsets.UTF_8);
settings.load(in);
```

또 다른 주의사항
: `Properties`는 제네릭이 없던 시절 만들어졌다 -> `Map(Object,Object)`를 기반으로 구현되었다.
- 부모인 `Map`이 가진 put/get 메서드를 호출하면 문자열이 아닌 다른 객체를 집어넣을 수 있다. -> 타입 에러 발생 가능
- `setProperty()`, `getProperty()`만을 사용한다는 규칙을 엄격히 지켜야한다.

직접 만든 설정 파일 외에도, JVM이나 OS가 시작될 때 자동으로 세팅하는 전역 환경 설정값들이 있다. 이 역시 `Properties` 형태로 관리된다.
- `System.getProperties()`: 시스템의 모든 환경 설정을 맵 형태로 가져온다.
- `System.getProperty("키 이름")`: 특정 설정값 하나만 문자열로 가져온다.
    - 예: `System.getProperty("user.home")` (현재 사용자의 홈 폴더 경로)    
    - 예: `System.getProperty("java.version")` (현재 실행 중인 자바 버전)

사용자가 설정 파일을 지웠거나, 특정 키가 누락되었을 때 프로그램이 뻗지 않게 하기위한 안전장치가 있다.
- 기본값 지정(못 찾을 경우 기본값 반환)
```java
String filename = settings.getProperty("filename", "default.txt");
```
- 기본값 전용 보조 테이블 : 두 번째 파라미터를 쓰지 않고, 기본값들만 모아둔 `Properties` 객체를 만든 뒤, 진짜 `Properties`를 만들 때 생성자에 넣는 방식
```java
// 시스템의 기본값을 세팅
var defaultSettings = new Properties();
defaultSettings.setProperty("width", "600");
defaultSettings.setProperty("height", "400");

// 이 기본값을 '보조 테이블'로 삼는 진짜 설정 객체를 만든다
var settings = new Properties(defaultSettings);

// 사용자가 width만 800으로 바꿨다고 가정해 보자
settings.setProperty("width", "800");

// width를 찾을 때:
System.out.println(settings.getProperty("width"));
// 결과: 800
// 이유: settings 본인의 맵에 800이 있으므로

// height를 찾을 때:
System.out.println(settings.getProperty("height"));
// 결과: 400
// 이유: settings 본인의 맵에는 height가 없다. 그래서 연결된 defaultSettings를 뒤져서 400을 찾아온다.

// color를 찾을 때:
System.out.println(settings.getProperty("color"));
// 결과: null
// 이유: 본인에게도 없고, 보조 테이블(defaultSettings)에도 없으므로 최종적으로 없다고 판단함
```


`Properties`의 한계와 대안.
- Properties는 디렉토리 구조처럼 Depth가 있지 않다. Flat 테이블임. 그래서 개발자들은 `window.main.color`처럼 변수 이름에 점을 사용해 계층 구조를 흉내낸다.
- 교재에서는 설정 정보가 이런 가짜 계층으로 감당이 안될 정도면, 10장에서 배울 `Preferences` 클래스를 사용하는 것이 올바른 아키텍처라고 권장한다.



## Stacks

`java.util.Stack`는 자바 1.0부터 존재한 후입선출 구조 클래스이다. 
Vector를 상속받았다. 그러나 vector가 리스트의 일종이라 스택의 중간에 값을 넣는것이 가능해졌다.
그렇기에 현재는 `Deque`인터페이스를 구현한 `ArrayDeque`를 사용하여 스택을 구현한다.


## Bit Sets

`java.util.BitSet` 은 참거짓을 나타내는 비트들의 배열이다. 
`boolean`값 하나를 저장하려면 1비트만 필요한 것이 상식적이다. 하지만 `ArrayList<Boolean>`을 쓰면 객체 오버헤드 때문에 `Boolean`객체 하나당 16바이트 이상의 메모리를 필요로하게 된다.

반면 `BitSet`은 내부적으로 여러 개의 비트를 하나의 큰 숫자(long 등) 단위로 처리하므로 메모리를 절약한다.

상태 조작:
- `set(i)`: $i$번째 비트를 1 (True)로 켭니다.        
- `clear(i)`: $i$번째 비트를 0 (False)으로 끕니다.    
- `get(i)`: $i$번째 비트가 1인지(True) 확인합니다.

크기 확인:    
- `cardinality()`: 현재 1(True)로 켜져 있는 비트의 총 개수를 반환합니다. (교재 예제에서 최종 소수의 개수를 셀 때 사용)
- `size()`: 내부적으로 할당된 전체 비트의 메모리 용량을 반환합니다.
- `length()`: 가장 마지막으로 켜져 있는 1의 위치 + 1 값(논리적 길이)을 반환합니다.
















