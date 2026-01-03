

단순히 데이터(`x`, `y` 좌표 등)만 담는 객체를 만드는데, 기존 `class`는 너무 번거로움.
데이터를 간결하게 정의
## Record Concept

A record is a special form of a class whose state is immutable and readable by the public.

```java
recode Point (double x, double y){
	...
}
//컴파일러가 자동으로 만들어 주는 것//
//private final instance fields (components)
	private final double x;
	private final double y;

//constructor
	Point(double x, double y);
//accessor method
	public double x()
	public double y()

//`toString()`, `equals()`, `hashCode()`

```

제약 사항 및 주의점
- **인스턴스 필드 추가 불가:** 레코드 헤더(소괄호 안)에 선언된 것 외에 별도 인스턴스 변수 선언 안 됨 (`private double r;` -> 에러).
    
- 가변 객체 주의: 레코드 필드는 `final`이지만, 필드가 가리키는 객체가 가변(예: `Date`)이면 내부 값은 바뀔 수 있음. 완벽한 불변을 원하면 필드 타입도 불변 객체를 써야 함.
	- `record PointInTime(double x, double y, Date when) { }`


## Constructors

**1. Canonical Constructor **
- 레코드 정의 시 선언한 모든 필드를 파라미터로 받는 자동 생성된 생성자.
- 별도로 작성 안 하면 컴파일러가 알아서 만들어 줌.
    
**2. Custom Constructor **
- 파라미터가 없는 생성자나, 특정 필드만 받는 생성자를 추가할 수 있음.
- **핵심 규칙:** 생성자 내부 첫 줄에서 **반드시 다른 생성자(`this(...)`)를 호출**해야 함.
- 결국 돌고 돌아 마지막엔 Canonical Constructor가 실행되어야 하기 때문임.
```java
recode Range(int from, int to)
{
	public Range(int from, int to)
	{
		if (from <= to)
		{
			this.from = from;
			this.to = to;
		}
		else
		{
			this.from = to;
			this.to = from;
		}
	}
}
```

**3. Compact Constructor**
- Canonical Constructor에 검증 로직이나 값 조작을 넣고 싶을 때 사용함.
- **특징:**
    - 메소드 파라미터 괄호 `()`를 아예 생략함 (`public Range { ... }`).
    - 필드 초기화 코드(`this.x = x`)를 작성하지 않음. 자동으로 수행됨.
    - **역할:** 필드에 값이 할당되기 전의 **'전처리(Prelude)'** 단계.
    - 내부에서 파라미터 변수(`from`, `to`) 값을 바꾸면, 그 바뀐 값이 필드에 저장됨.
    - **주의:** `this.from` 처럼 필드에 직접 접근하거나 수정 불가.
```java
record Range(int from, int to)
{
	public Range // compact form
	{
		if(from >to )
		{
			int temp = from;
			from = to;
			to =temp;
		}
	}
	//블록이 끝나면 자동으로 this.from = from; this.to = to; 가 실행됨 
	//(값을 미리 바꾸는것)
}
```


