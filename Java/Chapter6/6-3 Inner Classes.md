
내부 클래스 (Inner Class)?
- 다른 클래스 안에 정의된 클래스
사용하는 이유
- 내부 클래스는 같은 패키지 내의 다른 클래스로부터 숨겨질 수 있다. 
- 내부 클래스의 메서드는 자신이 정의된 외부 클래스 (Enclosing Class) 의 데이터(private 포함)에 직접 접근할 수 있다.

## Use of an Inner Class to Access Object State

```java
public class TalkingClock
{
    private int interval;
    private boolean beep; // private 변수

    public TalkingClock(int interval, boolean beep) { ... }
    public void start() { ... }

    // 내부 클래스 정의
    public class TimePrinter implements ActionListener
    {
        ...
    }
}
```
- 내부 클래스인 `TimePrinter`의 역할은 주기적으로 시간을 출력하고, 설정에 따라 비프음을 내는 것
```java
public class TimePrinter implements ActionListener
{
   public void actionPerformed(ActionEvent event)
   {
      System.out.println("At the tone, the time is "
         + Instant.ofEpochMilli(event.getWhen()));
      
      // TimePrinter에는 beep 변수가 없지만, 외부 클래스의 beep를 사용함
      if (beep) Toolkit.getDefaultToolkit().beep();
   }
}
```
- 이 `beep`는 자신을 생성한 `TalkingClock` 객체의 `beep` 필드이다. `beep`가 private임에도 접근이 가능

동작 원리
- 내부 클래스가 외부 클래스의 변수에 자신의 변수인 것 처럼 접근할 수 있는 이유: 암시적 참조(Implicit Reference)
- 내부 클래스의 객체가 생성될 때, 자바 컴파일러는 내부 클래스를 생성한 외부 클래스의 객체 주소를 내부 클래스안에 저장한다. (교재에서는 이를 `outer` 라고 지칭)
	- `if(beep)` -> `if(outer.beep)`
	- 내부 클래스에는 기본 생성자를 사용하지만, 컴파일러가 자동으로 생성자를 만들고 외부 객체를 받도록 조작. `public TimePrinter(TalkingClock clock) { outer = clock;}`
	- `start()` 에서 `new TimePrinter()` 를 호출할 때, 실제로는 현재 객체 (`this`)를 넘겨준다. `var listener = new TimePrinter(this);`

내부 클래스를 사용하는 이점 (Access control)
- `TimePrinter`를 일반 클래스로 만들었다면? 
	- `TalkingClock` 의 `beep` 변수를 외부에서 접근 불가능 했을 것이므로, 접근자 (getter)를 만들어야 했을 것이다. 
	- 하지만 이 접근자를 public으로 선언하면  `TimePrinter` 뿐만 아니라 다른 클래스들도 접근할 수 있게 되어 캡슐화가 깨진다.
- 내부 클래스를 사용하므로 `TimePrinter`에게만 접근 권한을 줄 수 있다. 또한, 내부 클래스 자체를 `private`로 선언하여 외부에서 아예 `TimePrinter`의 존재를 모르게 할 수 있음.


## Special Syntax Rules for Inner Classes

외부 클래스 참조 명시
- `OuterClass.this`
- 내부 클래스 안에서 `this`는 내부 클래스 자신의 객체를 지칭. 외부 클래스의 객체를 가리키고 싶을 때 사용
```java
public void actionPerformed(ActionEvent event){
	if(TalkingClock.this.beep){ //beep만 써도 되지만, 명확하게
		Toolkit.getDefaultToolkit().beep();
	}
}
```

내부 클래스 객체 생성
- 외부 클래스의 메서드 안에서는 단순히 `new InnerClass()`를 호출한다. 이때는 암시적으로 `this.new`가 적용된다. --> 새로 만들어진 내부 클래스 객체는 내부적으로 `this` 를 `outer`참조로 저장한다.
- 외부 클래스의 밖에서 내부 클래스의 객체를 생성하려면, 먼저 외부 클래스의 객체를 만들고, 그 객체를 통해 `new`를 호출해야 한다.
```java
// 1. 외부 클래스(TalkingClock) 객체 생성
var jabberer = new TalkingClock(1000, true);

// 2. jabberer 객체에 소속된 TimePrinter 객체 생성
// 참조변수.new 내부클래스()
TalkingClock.TimePrinter listener = jabberer.new TimePrinter();
```

내부 클래스 타입 참조
- 외부 클래스 밖에서 내부 클래스의 타입을 지칭할 때
- `TalkingClock.TimePrinter` 처럼 `.`을 사용해서 명시

제약 사항
- 내부 클래스 안에는 `static` 필드를 가질 수 없다.
- 단, `final`이면서 컴파일 타임 상수인 경우에는 허용
	- 상수가 아니라면, 내부 클래스 객체마다 static 필드를 따로 가질지, 공유할지 등의 개념이 모호해지고 유일성을 보장하기 어렵기 때문이다.
- 내부 클래스 안에는 `static` 메서드를 정의할 수 없다.


## Inner Classes UseFul? Actually Necessary?

컴파일러는 내부 클래스를 일반 클래스 파일로 변환한다.
- 클래스 파일 이름 : `외부클래스$내부클래스.class`
- `javap -private` 명령어로 컴파일러가 추가한 필드와 생성자를 볼 수 있다.
```java
// 컴파일 후 생성된 코드를 사람이 읽기 쉽게 복원한 모습
public class TalkingClock$TimePrinter implements ActionListener
{
    // 1. 외부 클래스를 가리키는 합성 필드 (Synthetic Field)
    final TalkingClock this$0; 

    // 2. 외부 클래스 참조를 받는 생성자
    public TalkingClock$TimePrinter(TalkingClock clock)
    {
        this.this$0 = clock; 
    }

    public void actionPerformed(ActionEvent event)
    {
        // ...
    }
}
```

개발자가 직접 일반 클래스로 만들어서 외부 객체를 전달받으면 되지 않나?
```java
class TimePrinter implements ActionListener {
    private TalkingClock outer;
    
    public TimePrinter(TalkingClock clock) {
        outer = clock;
    }

    public void actionPerformed(ActionEvent event) {
        // [문제 발생] 
        // 외부 클래스의 private 필드인 beep에 접근 불가!
        if (outer.beep) { ... } // ERROR!
    }
}
```
- 일반 클래스는 다른 클래스의 `private` 필드에 접근불가
- 내부 클래스는 접근 권한이 있다. --> 내부 클래스가 필요한 이유

내부 클래스의 외부 `private` 멤버에 접근 --> 캡슐화 위반 아닌가?
- Java 11 이전.
	- JVM은 내부 클래스에 대해서 모름. --> 컴파일러가 외부 클래스에 가짜 메서드(Synthetic Method) 를 만듦
```java
// Java 11 이전의 컴파일된 TalkingClock 클래스 내부
class TalkingClock
{
    private boolean beep;
    
    // 컴파일러가 몰래 만든 정적 접근 메서드
    static boolean access$0(TalkingClock outer)
    {
        return outer.beep;
    }
    // 내부 클래스가 beep을 필요로 하면 위의 메서드 호출 
    // 문제는 이 메서드의 존재를 알면(리플렉션 등을 통해), 
    //private인 beep변수를 무단으로 읽을 수 있다는 점이다.
}
```
- Java 11 이후.
	- JVM이 Nest 개념을 지원
	- JVM은 이 클래스들은 같은 Nest에 있다는 것을 인식한다. (`NestMembers` 속성)
		- --> 컴파일러가 더 이상 `access$0`같은 메서드를 만들지 않는다. 






























