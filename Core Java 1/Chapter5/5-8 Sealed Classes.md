상속을 원하는대로 통제하고 싶을 때 쓰는 도구

문제점
- 이전까지의 클래스 상속 방법
	- `final` 클래스 : 아무도 상속 받지 못한다.
	- 일반 클래스 : 누구나 상속 받을 수 있다.
- 원하는 곳에만 상속을 부여할 수 없음

해결책: `sealed` 클래스

사용법
- 부모 클래스: `sealed` , `permits`
	- `public abstract sealed class JSONValue permits JSONArray, JSONNumber, ...`
- 자식 클래스: 다음 셋 중 하나를 선택해서 명시해야 한다.
	- `final`, `sealed`, `non-sealed`(본인 부터 일반 클래스로 전환)

컴파일러가 자식 클래스의 종류를 알게 된다. -> switch 문에서 `default` 가 없어도 된다.
```java
public abstract sealed class JSONValue    
	permits JSONArray, JSONNumber, JSONString, JSONBoolean, JSONObject, JSONNull 
{    
	. . . 
}

...

public String type() {    
	return switch (this)    {       
		case JSONArray j -> "array";       
		case JSONNumber j -> "number";       
		case JSONString j -> "string";       
		case JSONBoolean j -> "boolean";       
		case JSONObject j -> "object";       
		case JSONNull j -> "null";       
		// No default needed here    
	}; 
}
```
예전 방식
```java
public String type() {
    if (this instanceof JSONArray) {
        return "array";
    } else if (this instanceof JSONNumber) {
        return "number";
    } else if (this instanceof JSONString) {
        return "string";
    } else if (this instanceof JSONBoolean) {
        return "boolean";
    } else if (this instanceof JSONObject) {
        return "object";
    } else if (this instanceof JSONNull) {
        return "null";
    } else {
        return "unknown"; // 혹시 몰라 넣어야 함
    }
}
```


