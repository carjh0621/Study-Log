
## predefined classes

math
`Math` class. You have seen that you can use methods of the Math class, such as `Math.random`, without needing to know how they are implemented—all you need to know is the name and parameters (if any).

That’s the point of encapsulation, and it will certainly be true of all classes. But the Math class only encapsulates functionality; it neither needs nor hides data. Since there is no data, you do not need to worry about making objects and initializing their instance fields —there aren’t any!

---
date
Date 클래스: 데이터(상태)를 가짐. 따라서 객체를 생성하여 인스턴스 필드를 초기화해야 사용할 수 있음.

use constructors to construct new instances.
Constructors always have the same name as the class name. Thus, the constructor for the Date class is called Date.

`new Date()` : This expression constructs a new object. The object is initialized to the current date and time.


`String s = new Date().toString();`  : That method yields a string representation of the date.

객체 변수 및 참조/참조 할당

- new 연산자: 생성자와 함께 사용되어 힙(Heap) 메모리에 새로운 객체를 만듦. 
	- `Date startTime;` → 아직 어떤 객체도 가리키지 않는 변수만 선언된 상태 (사용 시 컴파일 에러)
- `startTime = new Date();` → 새 객체를 만들어 그 참조값을 변수에 저장.
- `startTime = rightNow;` → 기존 객체를 가리키도록 설정 (두 변수가 하나의 객체를 공유).

You can explicitly set an object variable to null to indicate that it currently refers to no object.
```
startTime = null; . . .  
if (startTime != null)  
System.out.println(startTime);
```

## LocalDate

Date class, the time is represented by the number of milliseconds (positive or negative) from a fixed point, the so-called epoch, which is 00:00:00 UTC, January 1, 1970. UTC is the Coordinated Universal Time, the scientific time standard which is, for practical purposes, the same as the more familiar GMT, or Greenwich Mean Time.

Date class is not very useful for manipulating the kind of calendar information that humans use for dates, such as “December 31, 1999”.

Therefore, the standard Java library contains two separate classes: the `Date` class, which represents a point in time, and the `LocalDate` class, which expresses days in the familiar calendar notation.

You do not use a constructor to construct objects of the LocalDate class. Instead, use static factory methods that call constructors on your behalf.
- `LocalDate.now()`: 현재 날짜 객체 생성.
- `LocalDate.of(1999, 12, 31)`: 특정 날짜 객체 생성.

Once you have a LocalDate object, you can find out the year, month, and day  
with the methods getYear, getMonthValue, and getDayOfMonth:  
- `int year = newYearsEve.getYear(); // 1999`
- `int month = newYearsEve.getMonthValue(); // 12`  
- `int day = newYearsEve.getDayOfMonth(); // 31`

plusDays method yields a new LocalDate that is a  given number of days away from the object to which you apply it:  
```java
LocalDate aThousandDaysLater = newYearsEve.plusDays(1000);  
year = aThousandDaysLater.getYear(); // 2002  
month = aThousandDaysLater.getMonthValue(); // 09  
day = aThousandDaysLater.getDayOfMonth(); // 26
```
기존 객체의 날짜가 바뀌는 게 아니라 계산된 새로운 객체를 반환




