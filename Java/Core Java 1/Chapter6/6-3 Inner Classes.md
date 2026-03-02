
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



## Local Inner Classes

지금까지 본 `TimerPrinter` 같은 내부 클래스는 `TalkingClock` 클래스의 멤버(필드, 메서드)로 정의되었다. 
다만, 예제의 `TimerPrinter` 클래스는 오직 `start()` 메서드 안에서 객체를 생성할 때, 한번만 사용된다.
이런 경우, 클래스 정의 자체를 메서드 안으로 옮길 수 있다. 이를 지역 내부 클래스 (Local Inner Class) 라고 한다.

교제 예제
```java
public void start()
{
   // [지역 내부 클래스 정의]
   // 메서드 내부에서 클래스를 정의
   class TimePrinter implements ActionListener
   {
      public void actionPerformed(ActionEvent event)
      {
         System.out.println("At the tone, the time is "
            + Instant.ofEpochMilli(event.getWhen()));
         
         // 여전히 외부 클래스(TalkingClock)의 private 멤버인 beep에 접근 가능
         if (beep) Toolkit.getDefaultToolkit().beep();
      }
   }

   // 클래스가 정의된 바로 그 메서드 안에서만 사용 가능
   var listener = new TimePrinter();
   var timer = new Timer(interval, listener);
   timer.start();
}
```


특징
- 지역 클래스는 public, private, protected 와 같은 접근 제어자(Access Specifier) 를 붙일 수 없다.
	- 이 클래스는 해당 메서드 안에서만 존재하기 때문이다. 지역변수에 public을 붙이지 않는 것과 같음
- 지역 클래스의 스코프는 자신이 선언된 블록 내부로 엄격하게 제한된다.
	- `start()` 가 끝나면, 이 클래스는 더이상 코드 상에서 참조할 수 없게 된다.

장점
- 완벽한 은닉
	- 패키지 내의 다른 클래스는 물론이고, `TalkingClock` 클래스 안에 있는 다른 메서드들조차 `TimePrinter`의 존재를 알 수 없다.



## Accessing Variables from Outer Methods

시나리오
- 이전 예제에서는 `beep`가 `TalkingClock` 클래스의 멤버 변수였다. 
- 이번에는 `start`메서드의 지역 변수로 변경
```java
// beep는 이제 클래스 멤버가 아니라, start 메서드의 지역 변수
public void start(int interval, boolean beep)
{
   class TimePrinter implements ActionListener
   {
      public void actionPerformed(ActionEvent event)
      {
         // ...
         // 메서드의 지역 변수 beep를 사용
         if (beep) Toolkit.getDefaultToolkit().beep();
      }
   }
   
   var listener = new TimePrinter();
   var timer = new Timer(interval, listener);
   timer.start();
}
```

문제점
- Lifecycle 의 불일치
	- 1. `start(interval, beep)` 호출 --> `beep` 변수 생성
	- 2. `TimePrinter` 객체 생성
	- 3. `timer.start()` 호출 , `start` 종료 --> 지역변수 `beep` 이 스택 메모리에서 사라짐
	- 4. 1초 뒤, 타이머가 울리고 `actionPerformed` 실행 --> 이때 `if(beep)` 실행하려고 하는데, `beep` 변수는 이미 3번 단계에서 사라짐'
- --> `beep`을 어떻게 참조?

해결책: 변수 캡처 (Variable Capture)
- 자바 컴파일러는 지역 내부 클래스가 외부의 지역 변수를 사용하면, 그 변수의 값을 복사해서 내부 클래스 안에 저장해둔다.
```java
// 컴파일러가 실제로 변환한 코드
class TalkingClock$1TimePrinter
{
    // 1. 외부 클래스 참조
    final TalkingClock this$0;
    
    // 2.지역 변수 'beep'의 복사본을 저장할 필드 생성
    final boolean val$beep; 

    // 생성자에서 값을 받아와서 저장함
    TalkingClock$1TimePrinter(TalkingClock outer, boolean beep)
    {
        this.this$0 = outer;
        this.val$beep = beep; // 값을 복사
    }

    public void actionPerformed(ActionEvent event)
    {
        // 실제로는 원본 beep가 아니라, 복사된 val$beep를 사용
        if (val$beep) ... 
    }
}
```


제약 조건
- 복사 방식 때문에 제약이 생긴다. 
- 지역 내부 클래스에서 사용하는 지역 변수는 Effectively final 이어야 한다.
	- 만약 원본 변수(`beep`)의 값이 나중에 바뀐다면, 내부 클래스에 복사된 값(`val$beep`)과 서로 달라지게 된다(동기화 문제). 자바는 이런 혼란을 막기 위해 아예 값을 바꾸지 못하게 강제한다.



## Anonymous Inner Classes

Local Inner Class를 사용하다 보면, 클래스 이름조차 만들 필요가 없는 경우가 있다.
- 특정 클래스나 인터페이스를 상속하거나 구현하여 객체를 단 하나만 만들고 싶을 때
- --> 클래스 정의와 동시에 `new`로 객체를 생성

문법
```java
new SuperType(생성자 인수) 
{
    // 내부 클래스 메서드 및 데이터 정의
}
```
- `SuperType` 은 다음 두 가지 중 하나일 수 있다
	- 인터페이스 : `new ActionListener() {...}` --> 해당 인터페이스를 구현
	- 클래스: `new Person("Name") {...} `--> 해당 클래스를 상속

특징
- 생성자가 없음
	- 클래스에 이름이 없다 --> 명시적인 생성자를 작성할 수 없음
	- `new` 뒤에 오는 부모 클래스의 생성자에게 매개변수 전달 --> 인터페이스를 구현할 때는 생성자가 없으므로 괄호를 비워둠
- 초기화 블록
	- 생성자 대신 `{...}`를 사용하여 필드를 초기화할 수 있음.
- Lambda Expressions
	- `timer = new Timer(interval, event -> {...});`


## Static Inner Classes

일반적인 내부 클래스는 생성될 때 외부 클래스의 객체에 대한 hidden reference를 가진다. 하지만 때로는 이 참조가 불필요할 때가 있다.
- 어떤 클래스를 숨기거나, 이름 충돌을 방지하고 싶을 때 사용
- 내부 클래스 선언 앞에 `static` 붙으면 됨 --> 외부 클래스의 인스턴스 멤버에 접근 불가, 외부 클래스의 객체없이도 독립적으로 생성 가능

ex) 배열의 최소/최대값 구하기
- 배열을 두 번 훑는건 비효율적, 한 번만 훑어서 두개를 모두 반환하는게 좋다. 하지만, 자바의 메서드는 값 하나만 반환할 수 있다.
- 두 개의 값을 담아서 반환할 수 있는 클래스인 `Pair` 클래스
```java
// 값을 두 개 담을 수 있는 클래스
class Pair
{
   private double first;
   private double second;
   public Pair(double f, double s)
   {
      first = f;
      second = s;
   }
   public double getFirst() { return first; }
   public double getSecond() { return second; }
}
```
- 문제점
	- `Pair` 라는 이름이 너무 흔함. --> 충돌 발생 가능
- 해결책
	- `Pair` 클래스를 `ArrayAlg` 배열 알고리즘 클래스 안으로 넣으면 `ArrayAlg.Pair` 가 되어서 이름 충돌이 해결 가능
```java
class ArrayAlg
{
	public static class Pair
	{
		...
	}
	public static Pair minmax(double[] values) 
	{
		...
		return new Pair(min,max);
	}
}
```
- `minmax` 메서드는 `static`. `ArrayAlg`의 객체 없이 실행
- 만약 `Pair`가 일반 내부 클래스라면, `Pair` 객체를 만들기 위해 반드시 `ArrayAlg`객체가 필요하다.
- 하지만 `static`메서드 안에는 `this`가 없다. --> `Pair` 클래스도 `static`으로 선언해야만, 외부 객체 없이 `minmax` 메서드 안에서 생성될 수 있음.

특징
- 정적 내부 클래스의 객체는 자신을 생성한 외부 클래스 객체에 대한 참조를 가지지 않음. 따라서 메모리를 덜 사용하며, 외부 클래스의 생명주기와 무관하다.
- 일반 내부 클래스와 달리, 정적 내부 클래스는 static 필드와 static 메서드를 가질 수 있다.
- 인터페이스 내부에 선언된 클래스는 자동으로 `public` 이고 `static`이다.
- 클래스 내부에 선언된 인터페이스, 레코드, 열거형은 자동으로 `static`이다.










