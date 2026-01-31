
## The Assertion Concept

`double y = Math.sqrt(x);`에서 x가 음수가 아니라고 확신하는 것처럼, 코드를 작성하다 보면 특정 조건이 반드시 참이어야 한다고 확신하는 경우가 있다.
문제는 확신하더라도 만약의 경우를 대비해 검사를 하고 싶다는 것이다. 
`if (x<0) throw ...` 같은 예외 처리는, 배포용 코드에도 이 검사가 남아있어 프로그램 속도를 저하시킬 수 있다.
Assertion 메커니즘을 사용한다면, 테스트 중에는 검사를 수행하여 오류를 바로잡고, 배포할 경우 검사 코드가 무시되어 성능 저하가 없다.

Syntax
: Java에는 `assert`라는 키워드가 있다. 두 가지 형태가 있으며 두 경우 모두 조건이 false면 `AssertionError`를 던진다.
- `assert condition;` 
	- ex) `assert x >= 0;`
- `assert condition : expression;` 
	- ex) `assert x >= 0 : x;` 오류 발생 시 `x`의 값을 보여줌
	- `expression` 부분의 유일한 목적은 메시지 문자열을 생성하는 것이다. `AssertionError` 객체는 이 표현식의 원래 값을 저장하지 않는다. 이는 프로그래머가 실패한 단언문에서 복구하려고 시도하는 것을 방지하기 위함이다. (단언문 실패는 복구가 아닌 프로그램 수정이 필요한 상황이다.)
---

| 구분  | 복구 (Recovery)      | 수정 (Modification)                            |
| --- | ------------------ | -------------------------------------------- |
| 주체  | 프로그램 (예외 처리기)      | 개발자 (사람)                                     |
| 대상  | 예외적 상황 (Exception) | 버그 (Bug), 논리 오류                              |
| 대처  | `catch`해서 다른 길로 안내 | 코드를 고치고 재컴파일                                 |
| 예시  | 파일 없음, 네트워크 타임아웃   | NullPointer, IndexOutOfBound, AssertionError |
| 목표  | 서비스 중단 방지          | 결함 제거                                        |

---



## Assertion Enabling and Disabling

단언문의 가장 강력한 특징 중 하나는 소스코드를 수정하거나 재컴파일하지 않고도 실행 시점에 켜거나 끌 수 있다는 점이다. 이를 통해 개발/테스트 단계에서는 검증하고, 배포단계에서는 성능 저하 없이 실행할 수 있다.

Java 프로그램 실행 시 단언문은 기본적으로 비활성화 되어 있다. 
프로그램을 실행할 때 `-enableassertions` 또는 `-ea` 옵션을 사용한다
```
java -enableassertions MyApp //또는
java -ea MyApp
```

Class Loader Magic
: 단언문을 켜고 끄기 위해 프로그램을 다시 컴파일할 필요가 없다.
- 클래스 로더의 역할 : 단언문이 비활성화된 상태라면, 클래스 로더가 클래스 파일을 읽어들일 때 단언문 검사 코드를 아예 로딩하지 않고 제거한다. -> 따라서 단언문을 꺼두면 실행 속도가 느려지지 않는다.

Granular Control
: 옵션을 사용해서 특정 클래스나 패키지에 대해서만 단언문을 켜거나 끌 수 있다.
: 활성화
``` 
-ea:MyClass // 특정 클래스만
-ea:com.mycompany.mylib // 특정 패키지 및 하위 패키지, 패키지명 뒤에 점 3개
-ea... // 이름없는 패키지 전체
java -ea:MyClass -ea:com.mycompany.mylib... MyApp 
// MyClass 클래스와 com.mycompany.mylib 패키지 전체에서 단언문을 활성화
```
 : 비활성화
 ```
 java -ea... -da:MyClass MyApp
 // 모든 단언문을 켜되, MyClass 에서만 끈다
 ```


일반적인 `-ea`/`-da` 옵션은 클래스로더가 로드하는 클래스들에만 적용된다. JVM이 직접 로드하는 시스템 클래스에는 적용되지 않는다.
- `-enablesystemassertions` 또는 `-esa` 스위치를 사용

일부 프로그래머들은 클래스 파일의 크기를 줄이기 위해 단언문을 주석 처리하거나 조건부 컴파일 방식을 사용하기도 한다
```java
public static final boolean asserts = true; // 배포 시 false로 변경 후 재컴파일

...
if (asserts) assert x >= 0;
```
- `static final` 변수가 `false`면, 컴파일러 최적화 과정에서 `if`블록 내부 코드가 아예 생성되지 않는다.
- 하지만 현대의 JVM은 클래스 로더 수준에서 이를 효율적으로 처리하므로, 굳이 이렇게까지 할 필요는 많지 않다.



## Using Assertions for Parameter Checking


Java의 오류 처리 메커니즘
1. Throwing an exception: 복구 가능하거나 외부 사용자에게 알려야 할 때
2. Logging : 문제 상황을 기록으로 남길 때
3. Using assertions : 개발 단계에서 내부 버그를 잡을 때

단언문 선택 기준
: 단언문은 다음과 같은 특징을 가질 때 사용해야 한다. 
- Fatal, unrecoverable errors (치명적이고 복구 불가능한 오류)
- Dev/Test only (개발 및 테스트 중에만 켜져 있음)
- 즉, 실제 운영 중에는 성능을 위해 검사를 끄기 때문에, 복구해야하는 상황이나 사용자에게 알려야 하는 문제에는 단언문을 쓰면 안된다.

메서드 매개변수 검사
: ex) `sort` 메서드의 매개변수 검사에서 Contract(계약)에 따라 달라지는 선택 기준을 살펴보자
1.문서(Javadoc)에 예외가 명시된 경우
```java
/**
   @param a 정렬할 배열
   @param fromIndex 시작 인덱스
   @param toIndex 끝 인덱스
   @throws IllegalArgumentException 만약 fromIndex > toIndex라면
   @throws ArrayIndexOutOfBoundsException 만약 인덱스가 범위를 벗어난다면
*/
static void sort(int[] a, int fromIndex, int toIndex)
```
- 문서에는 인덱스가 잘못되면 예외를 던진다라고 Contract 하고 있다.
- 이 약속을 지키기 위해 반드시 예외를 던져야 한다.
- 따라서 `assert` 를 쓰면 안된다.

2.문서에 계약이 없는 경우
- 위의 경우에서 `a` 가 `null` 일 때 어떻게 된다는 말이 없을 때.
- 호출자는 `null`을 넣어도 메서드가 정상적으로(뭘 하든간에) 동작하기를 기대할 것이다.
- 그러므로 `assert a != null;` 을 넣으면 안된다. 이는 문서에 없는 제약을 멋대로 추가하는 행위기 때문이다.

3.Precondition 이 명시된 경우
```java
/**
   @param a 정렬할 배열 (단, null이어서는 안 됨)
*/
```
- 문서가 호출자는 `null`을 넘겨서는 안된다라고 경고하고 있음. 이를 사전 조건이라고 함.
- 만약 `null`이 들어오면, 그것은 호출자의 잘못 = 호출자 코드의 버그이다.
- 이때는 `assert a != null;` 을 사용할 수 있다.



## Using Assertions for Documemting Assumptions

주석 대신 단언문 사용
: 기존 방식
``` java
if (i % 3 == 0)
   . . .
else if (i % 3 == 1)
   . . .
else // (i % 3 == 2)  <-- 주석으로 가정을 설명함
   . . .
```
- 주석은 실행되지 않으므로, 만약 코드가 수정되어 가정이 틀리게 되더라도 (`i`가 음수일 때) 알 수가 없다.
: 단언문
```java
if (i % 3 == 0)
   . . .
else if (i % 3 == 1)
   . . .
else
{
   assert i % 3 == 2; // <-- 코드로 검증함
   . . .
}
```
- 이제 이 가정은 테스트 시마다 검증된다. 의도한 상황이 아니면 `AssertionError` 가 발생한다.


java.lang.ClassLoader
: 단언문 활성화/비활성화는 클래스로더를 통해 프로그램 코드 내에서도 제어할 수 있다.

| 메서드                                                        | 설명                                                             |
| ---------------------------------------------------------- | -------------------------------------------------------------- |
| `setDefaultAssertionStatus(boolean b)`                     | 이 클래스 로더가 로드하는 모든 클래스에 대해 기본적으로 단언문을 켜거나 끈다. (개별 설정이 없는 경우 적용) |
| `setClassAssertionStatus(String className, boolean b)`     | 특정 클래스(및 그 내부 클래스)에 대해 단언문을 켜거나 끈다.                            |
| `setPackageAssertionStatus(String packageName, boolean b)` | 특정 패키지와 그 하위 패키지의 모든 클래스에 대해 단언문을 켜거나 끈다.                      |
| `clearAssertionStatus()`                                   | 모든 명시적 설정을 지우고, 기본적으로 모든 클래스의 단언문을 비활성화.                       |




























