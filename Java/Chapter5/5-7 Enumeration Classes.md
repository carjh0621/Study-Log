
Enum은 클래스이다
- `public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }`
- Size 라는 클래스, 안에 SMALL, MEDIUM 같은 객체가 4개만 미리 만들어져 있는 구조.

비교
- `==` 쓰면 된다. 
- `equals()` 안써도 됨. 
- 객체가 딱 하나뿐이라, 주소값 비교를 해도 안전

교안 예제
```java
public enum Size
{
	SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");    
	private String abbreviation;    
	Size(String abbreviation) { this.abbreviation = abbreviation; }        
	// automatically private    
	public String getAbbreviation() { return abbreviation; }
}
```
상수마다 값을 부여하거나 기능을 넣을 수 있음.
- `SMALL("S")`, `MEDIUM("M")` 처럼 괄호 안에 값을 넣어서 생성자를 호출함.
- `abbreviation`이라는 필드에 "S", "M" 같은 약어를 저장해둠.
- 생성자 주의사항
	- 무조건 `private` (자동으로 private)
	- 프로그램 시작할 때 상수가 만들어지면서 자동으로 호출됨. `public` 선언 시 에러

모든 열거형은 자동으로 `Enum` 클래스를 상속받는다.
--> 다른 클래스를 상속받을 수 없음

내장 메서드:
- `toString()`: 상수 이름을 문자열로 반환 (예: "SMALL").
- `valueOf(Class, String)`: 문자열을 다시 Enum 객체로 바꿈 (예: "SMALL" → `Size.SMALL`).
- `values()`: 모든 상수를 배열로 리턴함 (반복문 돌릴 때 씀).
- `ordinal()`: 상수의 순서(인덱스)를 0부터 반환함 (예: SMALL=0, MEDIUM=1).


