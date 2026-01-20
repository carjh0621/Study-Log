 애플리케이션 개발 시, 서비스 아키텍처 패턴을 사용하는 경우가 많다.
 이는 서비스의 설계와 구현을 분리하여 개발자가 나중에 구현체를 자유롭게 교체하거나 선택할 수 있게 하는 방식
 - 공통 인터페이스를 정의하고 이를 구현한 다양한 클래스를 동적으로 로드하여 사용하도록 한다.


교재 Cipher 예제
- 1. 서비스 인터페이스 정의
	- 모든 서비스 인스턴스가 제공해야할 기능을 정의한 인터페이스(또는 상위 클래스)를 만든다
```java
package serviceLoader;

public interface Cipher{
	byte[] encrypt(byte[] source, byte[] key);
	byte[] decrypt(byte[] source, byte[] key);
	int strength();
}
```
- 2. 서비스 구현체 작성
	- 위 인터페이스를 구현하는 클래스를 하나 이상 만든다. 구현체는 인터페이스와 같은 패키지에 있을 필요가 없다. 
	- 다만, 각 구현 클래스는 no-argument constructor를 가져야 한다.
```java
package serviceLoader.impl;

import serviceLoader.Cipher;

public class CaesarCipher implements Cipher {
    // 인자가 없는 기본 생성자가 암시적으로 존재함 (public 이어야 함)
	public CaesarCipher() { 
		// ServiceLoader는 반드시 public 기본 생성자가 필요함
	}
    public byte[] encrypt(byte[] source, byte[] key) {
        var result = new byte[source.length];
        for (int i = 0; i < source.length; i++) {
            // 간단한 시저 암호: 원본 바이트에 키 값을 더함
            result[i] = (byte)(source[i] + key[0]);
        }
        return result;
    }

    public byte[] decrypt(byte[] source, byte[] key) {
        // 복호화: 암호화의 역연산 (키 값을 뺌)
        return encrypt(source, new byte[] { (byte) -key[0] });
    }

    public int strength() { return 1; }
}
```
- 3. 서비스 설정 파일 (META-INF)
	- ServiceLoader가 구현 클래스를 찾을 수 있도록 설정 파일을 만들어야 하낟.
	- 디렉토리: `META-INF/services`
	- 파일명: 인터페이스의 완전한 이름 (Fully qualified name): `serviceLoader.Cipher`
	- 파일 내용: 구현 클래스의 완전한 이름, 인코딩은 UTF-8
`META-INF/services/serviceLoader.Cipher` : `serviceLoader.impl.CaesarCipher`


사용방법
- 1. 초기화: 프로그램 내에서 한번만 수행하면 됨
```java
public static ServiceLoader<Cipher> cipherLoader = ServiceLoader.load(Cipher.class);
```
- 2. 반복자를 이용한 사용
	- `ServiceLoader`는 `Iterable` 인터페이스를 구현하고 있음. 
```java
public static Cipher getCipher(int minStrength) {
    // cipherLoader를 순회할 때마다 구현 클래스가 지연 로딩(Lazily loaded)됨
    for (Cipher cipher : cipherLoader) { // 암시적으로 iterator() 호출
        if (cipher.strength() >= minStrength) {
            return cipher;
        }
    }
    return null;
}
```
- 3. 스트림을 이용한 사용
	- Java 9부터는 `stream()` 메서드를 통해 `ServiceLoader.Provider` 인스턴스의 스트림을 얻을 수 있다.
	- `Provider.type()`을 통해 클래스를 인스턴스화 하지 않고도 구현 클래스의 타입을 검사할 수 있다.
```java
import java.util.Optional;
import java.util.ServiceLoader;

public static Optional<Cipher> getCipher2(int minStrength) {
    return cipherLoader.stream()
        .filter(descr -> descr.type() == serviceLoader.impl.CaesarCipher.class) // 타입으로 먼저 필터링
        .findFirst()
        .map(ServiceLoader.Provider::get); // 실제로 필요할 때 인스턴스 생성 (get())
}
```

주요 API

| **메서드**                                              | **버전** | **설명**                                                              |
| ---------------------------------------------------- | ------ | ------------------------------------------------------------------- |
| `static <S> ServiceLoader<S> load(Class<S> service)` | 1.6    | 주어진 서비스 인터페이스를 구현하는 클래스를 로드하기 위한 로더를 생성                             |
| `Iterator<S> iterator()`                             | 1.6    | 서비스 클래스들을 지연 로딩(Lazily load)하는 반복자를 반환한다. 반복자가 다음으로 넘어갈 때 클래스가 로드됨. |
| `Stream<Provider<S>> stream()`                       | 9      | `Provider` 디스크립터의 스트림을 반환. 이를 통해 원하는 클래스인지 확인 후 로드 가능               |
| `Optional<S> findFirst()`                            | 9      | 사용 가능한 첫 번째 서비스 제공자를 찾는다. (없을 수 있으므로 Optional 반환)                   |

---

폴더 구조
```
MyProject/
 └── src/
      ├── META-INF/
      │    └── services/
      │         └── serviceLoader.Cipher  <-- 설정 파일
      │
      ├── serviceLoader/              <-- 패키지 1: 인터페이스 정의
      │    └── Cipher.java            <-- (인터페이스)
      │
      ├── serviceLoader/impl/         <-- 패키지 2: 실제 구현체들
      │    └── CaesarCipher.java      <-- (구현 클래스)
      │
      └── client/                     <-- 패키지 3: 사용자 (Main이 여기 있다고 가정)
           └── App.java               <-- (여기에 main 함수가 있음)
```


`ServiceLoader.load(Cipher.class)`
1. **Main(App.java):** `ServiceLoader`에게 `serviceLoader.Cipher`라는 인터페이스(규격)를 따르는 것들을 찾아달라 요청 (`load` 호출)
2. **ServiceLoader:**  `serviceLoader.Cipher`라는 이름의 설정 파일이 있는지 폴더(`META-INF/services/`)를 찾아봄.
3. **탐색:** `META-INF/services/` 폴더로 가서 `serviceLoader.Cipher`라는 이름의 파일을 연다.
4. **발견:** 그 파일 안에 적힌 글자를 읽음.
    - 파일 내용: `serviceLoader.impl.CaesarCipher`
5. **로딩:**  `serviceLoader.impl.CaesarCipher` --> 자바는 그제야(지연 로딩) `CaesarCipher` 클래스를 메모리에 올리고 객체(instance)를 생성할 준비를 한다.

---






























