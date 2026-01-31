
저자가 제시하는 8가지 팁

1.Exception handling is not supposed to replace a simple test.
: 예외 처리는 정상적인 제어 흐름을 대체하기 위해 설계된 것이 아니며, 성능 면에서도 비효율적이다.
```java 
// 나쁜 예시
try {
    s.pop();
} catch (EmptyStackException e) {
}

// 좋은 예시 - 사전에 상태를 검사
if (!s.empty()) s.pop();
```
- 저자의 테스트에서 단순 검사에 비해 예외 처리 방식은 훨씬 시간이 오래 걸렸다.

2.Do not micromange exceptions
: 모든 문장을 개별적인 try 블록으로 감싸는 것은 코드를 비대하게 만들고 가독성을 떨어뜨린다.
```java
// 나쁜 예시 - 반복분 내에서 매번 예외 처리수행
for (i = 0; i < 100; i++) {
    try { n = s.pop(); } catch (EmptyStackException e) { ... }
    try { out.writeInt(n); } catch (IOException e) { ... }
}

// 좋은 예시 - 전체 Task를 하나의 try 블록으로 감쌈
for (i = 0; i < 100; i++) {
    try { n = s.pop(); } catch (EmptyStackException e) { ... }
    try { out.writeInt(n); } catch (IOException e) { ... }
}
```


3.Make good use of the exception hierarchy
- 단순히 `RuntimeException` 보다는 적절한 하위 클래스를 찾거나 직접 만들기
- `catch (Throwable t)` 는 코드를 유지보수하기 어렵게 한다.
- Logic error 에는 Checked Exception을 던지지 말 것
- `NumberFormatException` 을 잡아서 상황에 더 적합한 `IOException`이나 사용자 정의 예외로 변환하여 던지는 것을 망설이지 말 것

4.Do not squelch exceptions
:  컴파일러 경고를 없애기 위해 비어있는 `catch` 블록을 만들지 말 것
- 예외를 무시하면 프로그램은 문제없이 실행되는 것처럼 보이지만, 오류 발생 시 아무런 조치 없이 조용히 실패하게 된다. 예외가 중요하다면 반드시 적절히 처리해야 한다.

5.When you detect an error, "tough love" works better than indulgence
: 오류가 발생했을 때 더미 값을 반환하는 것보다, 즉시예외를 던지는 것이 낫다.
- 스택이 비었을 때 `null` 을 반환하면, 나중에 이상한 곳에서 `NullPointerException` 이 발생하여 원인을 찾기 어려워진다. 차라리 처음부터 `EmptyStackException`을 던지는 것이 디버깅에 유리하다.

6.Propagating exceptions is not a sign of shame
: 상위 레벨의 메서드가 사용자에게 오류를 알리거나 작업을 취소하는 등의 처리를 하기에 더 적합한 경우가 많다.
- 어설프게 catch 하는것보다, 메서드 헤더에 `throws`를 명시하여 호출자에게 떠넘기는 것이 더 낫다.

7.Use standard methods for reporting null-pointer and out-of-bounds exceptions
: `Objects` 클래스의 메서드를 사용하여 매개변수 유효성을 검사하면 표준화된 오류 메시지를 사용할 수 있다.
- `Objects.requireNonNull(newValue)`
- `Objects.checkIndex(position, data.length)`
- `Objects.checkFromToIndex` 
- 등등

8.Don't show stack traces to end users
: stack trace는 디버깅에 필수적이지만, 사용자에게 그대로 보여주는 것은 좋지 않다.
- 사용 중인 라이브러리 버전 등의 내부 구현 정보가 노출되어 해커의 표적이 될 수 있다.
- --> 스택 추적은 로그 파일에 남기고, 사용자에게는 이해하기 쉬운 요약 메시지만 보여주는 것이 바람직하다.
























