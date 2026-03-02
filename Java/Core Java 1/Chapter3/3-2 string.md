## string 
``` java
String e = ""; // an empty string 
String greeting = "Hello";
```

## substring
```java
String greeting = "Hello"; 
String s = greeting.substring(0, 3);
//creates a string consisting of the characters "Hel".
```

## Concatenation
```java
String expletive = "Expletive"; 
String PG13 = "deleted"; 
String message = expletive + PG13;
// variable message to the string "Expletivedeleted"
```

```java 
int age = 13; 
String rating = "PG" + age;
//sets rating to the string "PG13".
```

```java
System.out.println("The answer is " + answer);
//perfectly acceptable and prints what you would expect
String all = String.join(" / ", "S", "M", "L", "XL");    
// all is the string "S / M / L / XL"
String repeated = "Java".repeat(3); 
// repeated is "JavaJavaJava"
```

## string are immutable

If you want to turn greeting into "Help!", you cannot directly change the last positions of greeting into 'p' and '!'
-> Concatenate the substring that you want to keep with the characters that you want to replace.
``` java
String greeting = "Hello"; 
greeting = greeting.substring(0, 3) + "p!";
//greeting variable to "Help!"
```

immutable strings have one great advantage: The compiler can arrange that strings are shared.
- To understand how this works, think of the various strings as sitting in a common pool. String variables then point to locations in the pool. If you copy a string variable, both the original and the copy share the same characters.
- Overall, the designers of Java decided that the efficiency of sharing outweighs the inefficiency of string editing by extracting substrings and concatenating. Look at your own programs; most of the time, you probably don’t change strings—you just compare them.
## test strings
```java
"Hello".equals(greeting) 
//returns true if the strings s and t are equal, false otherwise.
"Hello".equalsIgnoreCase("hello")
//To test whether two strings are identical except for the upper/lowercase letter distinction
```
Do not use the == operator to test whether two strings are equal! It only determines whether or not the strings are stored in the same location. 
- Sure, if strings are in the same location, they must be equal. 
- But it is entirely possible to store multiple copies of identical strings in different places.
```java
String greeting = "Hello"; // initialize greeting to a string  
if (greeting == "Hello") . . .  
        // probably true  
if (greeting.substring(0, 3) == "Hel") . . .  
// probably false
```

## empty, null string
The empty string "" is a string of length 0. You can test whether a string is empty by calling
``` java
if (str.length() == 0)  
//or
if (str.equals(""))  
```
An empty string is a Java object which holds the string length (namely, 0) and an empty contents. 
However, a String variable can also hold a special value, called null, that indicates that no object is currently associated with the variable. (See Chapter 4 for more information about null.) To test whether a string is null, use

```java
if (str == null)  
//Sometimes, you need to test that a string is neither null nor empty. Then use  
if (str != null && str.length() != 0)
```

## Code Points and Code Units

char data type is a code unit for representing Unicode code points in the UTF-16 encoding.
The most commonly used Unicode characters can be represented with a single code unit. The supplementary characters require a pair of code units.
### Code Point (코드 포인트)
- **“하나의 문자”**
- Unicode에서 문자를 식별하는 정수 값
- 예:
    - `'A'` → U+0041   
    - `'가'` → U+AC00
    - `𝕆` → **U+1D546**
- **사람이 인식하는 문자 단위**
### Code Unit (코드 유닛)
- **UTF-16에서 문자를 저장하는 최소 단위**
- Java에서는 **`char` = code unit**
- 크기: **16비트**
- **컴퓨터가 실제로 저장하는 단위**

### UTF-16의 특징
- 대부분의 문자 → **code unit 1개**
- 일부 문자(보조 평면, emoji, 수학 기호 등) → **code unit 2개**
- 이 **2개짜리 문자**를  **supplementary character** 라고 부름

예제: 
-  `𝕆`
	- Unicode: **U+1D546**
	- UTF-16에서는?
    - **code unit 2개 (surrogate pair)** 
- `𝕆 is the set of octonions.`
- `char ch = sentence.charAt(1);` 
	- 결과:  공백이 아닌 **𝕆의 두 번째 코드 유닛**
- `sentence.length();`
	- - ❌ 문자 개수
	- ✅ **code unit 개수**

해결:
- `int index = s.offsetByCodePoints(0, i);
- `int cp = s.codePointAt(index);`
	- i번째 code point가 시작되는 code unit의 위치
	- 그 위치에서 code point 전체를 읽음
- `int cpCount = sentence.codePointCount(0, sentence.length());`
	- code point 기준 길이


## string API

https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html

## building string

Every time you concatenate strings, a new String object is constructed. This is time consuming and wastes memory. Using the StringBuilder class avoids this problem
```java
StringBuilder builder = new StringBuilder();

builder.append(ch); // appends a single character
builder.append(str); // appends a string

String completedString = builder.toString();
//When you are done building the string, call the toString method. You will get a String object with the character sequence contained in the builder.
```

toString()을 호출해도 builder는 그대로 남아 있음

## text blocks

```java
String s = """
Hello
World
""";
// "Hello\nWorld\n"
```

```java
String html = """
<div class="Warning">
   Beware of those who say "Hello" to the world
</div>
""";
// "는 그냥 문자
```

따옴표를 escape 해야하는 경우
- 문자열이 `"`로 끝나는 경우. 마지막 "는 `\"` 로
-  `"""`가 문자열 안에 등장하는 경우 마찬가지로 \ 이용

``` java
"""
C:\Users\Name
"""
(x) 
"""
C:\\Users\\Name
"""
(o)
```

```java
"""
Hello, my name is Hal. \
Please enter your name: 
"""
//"Hello, my name is Hal. Please enter your name: "
```

Java는 text block에서:
- 줄 끝 공백 제거
- `\r\n` → `\n` 으로 통일
그래서 만약 **줄 끝 공백을 꼭 보존**해야 하면:

``` java
""" 
Hello\s 
"""
//`\s` = space 1칸
```

```java
String html = """
    <div>
        Hello
    </div>
""";
/* 실제 문자열 (모든 줄에 공통으로 있는 공백/탭을 제거)
<div>
    Hello
</div>
*/
```

