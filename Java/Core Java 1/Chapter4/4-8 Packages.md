
클래스들을 묶어서 관리하는 그룹
클래스 이름 충돌 방지를 위해서
## Packages name

전 세계적으로 유일한 이름을 갖기 위해 인터넷 도메인을 거꾸로 뒤집어서 사용함.
- `horstmann.com` 소유자라면 -> `com.horstmann`으로 시작.
- 그 뒤에 프로젝트명을 붙임 -> `com.horstmann.corejava`.
- 그러면 Fully qualified name 은 `com.horstmann.corejava.Employee`가 됨.

패키지 이름이 계층적(`java.util` 안에 `java.util.jar`)으로 보인다고 해서 진짜 포함 관계인 건 아님    
- 컴파일러 입장에서는 둘은 그냥 **완전히 별개의 패키지**임 (서로 아무 관계 없음).
## Class Importation

You can access the public classes in another package in two ways. The first is simply to use the fully qualified name; that is, the package name followed by the class name. For example:
- `java.time.LocalDate today = java.time.LocalDate.now();`

The point of the import statement is to give you a shorthand to refer to the classes in the package. Once you add an import, you no longer have to give the classes their full names.
- `import java.time.*;` 
- Then you can use `LocalDate today = LocalDate.now();`

**이름 충돌 (Name Conflicts) 해결**

- 예를 들어 `java.util.*`과 `java.sql.*`을 둘 다 임포트 했는데, 둘 다 `Date` 클래스가 있음    
- 그냥 `Date`라고 쓰면 컴파일러가 누굴 말하는지 몰라서 에러 남.
    
- **해결법 1 (우선순위 지정):** 더 자주 쓰는 쪽을 콕 집어서 임포트 함.
    - `import java.util.Date;`를 추가하면 `Date`는 `util` 패키지 걸로 인식됨.
        
- **해결법 2 (둘 다 써야 할 때):**
    - 어쩔 수 없이 코드 내에서 **풀네임**을 써야 함.
    - `var a = new java.util.Date();`
    - `var b = new java.sql.Date(...);`


## Static Import

일반적인 `import`는 클래스를 가져오지만, `import static`은 클래스 안의 **static 메소드나 필드**를 바로 가져오는 기능

```java
import static java.lang.System.*;
//or import static java.lang.System.out;

//then
out.println(...); //i.e., System.out
exit(0); // i.e., System.exit
```


## Class into a Package

소스 파일 가장 첫 줄에 `package 패키지명;`을 적으면 됨.
이걸 안 적으면 Unnamed Package 에 들어감

**패키지 이름과 디렉토리 경로는 무조건 똑같아야 함.**    
- 예: `package com.horstmann.corejava;`
    - 윈도우: `com\horstmann\corejava\Employee.java`
    - 맥/리눅스: `com/horstmann/corejava/Employee.java`
- 컴파일러는 파일(`src/com/...`)을 보고, JVM은 클래스(`com.horstmann...`)를 로딩함
```
BaseDir/
 ├── PackageTest.java           (패키지 선언 없음 = default package)
 └── com/
      └── horstmann/
           └── corejava/
                └── Employee.java  (package com.horstmann.corejava)
```

베이스 디렉토리(Base Directory)에서 명령어를 쳐야 함.
- 베이스 디렉토리란? 패키지 최상위 폴더(`com`)가 있는 그 상위 폴더.

**컴파일 (`javac`):** 파일 경로(슬래시 `/`)를 사용.    
    - `javac com/mycompany/PayrollApp.java`
        
**실행 (`java`):** 클래스 이름(점 `.`)을 사용.
    - `java com.mycompany.PayrollApp`

## Package Access

- **접근 제어자**
    - `public`: 어디서든 접근 가능.
    - `private`: 해당 클래스 내부에서만 접근 가능.
    - Default: 아무것도 안 쓰면 같은 패키지 안의 모든 클래스가 접근 가능함.
        
- **문제점 (캡슐화 깨짐):**
    - 변수(필드)에 실수로 `private`을 빼먹으면, 같은 패키지 내 다른 클래스가 마음대로 값을 바꿀 수 있음.
    - 예시: JDK의 `java.awt.Window` 클래스에 `warningString`이라는 필드가 `private`이 아님.


## Class Path

JVM이나 컴파일러가 클래스 파일(`*.class`)을 찾기 위해 뒤지는 경로들의 집합
일반 디렉토리에 있거나, JAR 파일(여러 클래스를 압축한 ZIP 파일) 안에 들어있음

**클래스 패스 설정 방법**
- 필요한 경로(기본 디렉토리, JAR 파일 등)를 다 나열해야 함.
- 운영체제별 구분자(Separator) 차이:
    - **Windows:** 세미콜론 (`;`) 사용
        - 예: `c:\classdir;.;c:\archives\archive.jar`
            
    - **UNIX (Linux/Mac):** 콜론 (`:`) 사용
        - 예: `/home/user/classdir:.:/home/user/archives/archive.jar`
            
- **와일드카드 사용:** Java 6부터 `디렉토리/*`를 쓰면 그 폴더 안의 모든 JAR를 포함시킬 수 있음.


JVM의 클래스 검색 순서
1. **Java API:** 일단 JDK에 내장된 기본 라이브러리부터 찾음 (설정 안 해도 됨).
2. **클래스 패스:** 위에서 설정한 경로들을 앞에서부터 순서대로 뒤짐.
    - 예: `com.horstmann.corejava.Employee`를 찾는다면, 패스에 등록된 경로 아래에 `com/horstmann/corejava/Employee.class`가 있는지 확인.
        

컴파일러(javac)의 특징
- 컴파일러는 JVM보다 할 일이 더 많음 (`import` 구문도 해결해야 하니까).
- 소스 파일 재컴파일 기능
    - 클래스 파일(`.class`)을 찾았더라도, 원본 소스 파일(`.java`)이 더 최신이면 자동으로 다시 컴파일함.
        
- 소스 파일 검색:
    - `public` 클래스는 파일명과 클래스명이 같아서 찾기 쉬움.
    - 같은 패키지 내의 `non-public` 클래스는 해당 패키지의 모든 소스 파일을 뒤져서 정의를 찾아냄.
        

**주의사항 (CAUTION)**
- **현재 디렉토리 점(.)의 중요성:**
    - 클래스 패스를 아예 설정 안 하면 기본값이 `.`(현재 경로)라서 문제없음.
    - 하지만 클래스 패스를 따로 설정할 때는 반드시 `.`(현재 디렉토리)를 포함시켜야 함.
    - 안 그러면, `javac`는 현재 폴더 파일을 보는데, 정작 실행(`java`)할 때 JVM이 현재 폴더를 안 봐서 못 찾고 에러 남.


## Setting Class Path

실행할 때마다 `-classpath` (또는 `-cp`, Java 9부터는 `--class-path`) 옵션을 사용
- **장점:** 각 프로그램마다 필요한 경로를 명확하게 지정할 수 있어서 다른 프로그램이랑 충돌 안 남

운영체제의 환경 변수에 `CLASSPATH`를 등록해두는 방법

**단점:**
- **전역 설정의 위험:** 한번 설정해두면 모든 자바 프로그램이 이 경로를 참조함. A 프로그램용 라이브러리가 B 프로그램 실행을 방해할 수 있음.
    
- **실수 유발:** 예전에 Apple QuickTime 설치 프로그램이 맘대로 전역 `CLASSPATH`를 바꿨는데, 현재 디렉토리(`.`)를 빼먹어서 전 세계 개발자들 코드가 실행 안 되는 대참사가 있었음.

**Linux/Mac (구분자: 콜론 `:`)**

```bash
# 현재 디렉토리(.)와 라이브러리 경로, jar 파일을 모두 포함
java -cp /home/user/classdir:.:/home/user/archives/archive.jar MyProg
```




# 가상의 프로젝트 예시

우리가 만들 프로젝트 이름은 `MyTinyWebServer`라고 하자. 기능별로 패키지를 3개
- `com.myserver.main`: 실행 진입점 (Main)
- `com.myserver.core`: 서버 핵심 로직 (Server)
- `com.myserver.http`: HTTP 요청/응답 처리 (Request, Response)
    

**폴더 구조:**
```
MyTinyWebServer/             <-- 프로젝트 루트 (여기서 터미널 염)
├── bin/                     <-- 컴파일된 .class 파일들이 모일 곳 (비워둠)
└── src/                     <-- 소스 코드
    └── com
        └── myserver
            ├── main
            │   └── Main.java         (Server 클래스를 사용함)
            ├── core
            │   └── Server.java       (HttpRequest 클래스를 사용함)
            └── http
                ├── HttpRequest.java
                └── HttpResponse.java
```

---

### 2. 소스 코드 의존 관계 (간략)

컴파일 순서를 이해하려면 누가 누구를 쓰는지 알아야 함.

1. **`Main.java`**
```java
    package com.myserver.main;
    import com.myserver.core.Server; // core 패키지 사용
    
    public class Main {
        public static void main(String[] args) {
            new Server().start();
        }
    }
    ```
    
2. **`Server.java`**
  ``` java
    package com.myserver.core;
    import com.myserver.http.HttpRequest; // http 패키지 사용
    
    public class Server {
        public void start() {
            System.out.println("서버 시작...");
            HttpRequest req = new HttpRequest(); // 의존성 발생
        }
    }
    ```
    

---

### 3. 컴파일 방법 (Compile)

이제 터미널을 열고 **프로젝트 루트(`MyTinyWebServer`)**로 이동함.

``` bash
javac -d bin -sourcepath src src/com/myserver/main/Main.java
```

옵션
- **`-d bin`**: "컴파일된 `.class` 파일들을 `bin` 폴더에다가 패키지 구조(`com/myserver/...`) 그대로 만들어서 넣어라." (이거 안 하면 `src` 폴더에 `.class`가 섞여서 지저분해짐) 
- **`-sourcepath src`**: "혹시 `Main.java` 컴파일하다가 모르는 클래스 나오면 `src` 폴더 뒤져서 찾아라."
- **`src/com/myserver/main/Main.java`**: 컴파일할 대상 파일.
    
**Q. 왜 `Main.java` 하나만 컴파일 함? 나머지는?**

- A. 자바 컴파일러(`javac`)는 똑똑함. `Main`을 컴파일하려는데 `Server`가 필요하면 알아서 `Server.java`를 찾아서 컴파일하고, `Server`가 `HttpRequest`를 쓰면 걔도 찾아서 컴파일함. 

---
### 4. 컴파일 결과 확인

컴파일이 끝나면 `bin` 폴더가 이렇게 자동으로 채워짐.

```
MyTinyWebServer/
├── bin/
│   └── com
│       └── myserver
│           ├── main
│           │   └── Main.class
│           ├── core
│           │   └── Server.class
│           └── http
│               ├── HttpRequest.class
│               └── HttpResponse.class
└── src/ ...
```

---

### 5. 실행 방법 (Run)

여전히 터미널 위치는 **프로젝트 루트(`MyTinyWebServer`)**임.
``` bash
java -cp bin com.myserver.main.Main
```

옵션 

- **`-cp bin` (또는 `-classpath bin`)**: 클래스 파일 찾을 때 `bin` 폴더를 기준(Base Directory)으로 찾기
    
- **`com.myserver.main.Main`**: 실행할 클래스의 **풀네임(Fully Qualified Name)**.
    - 절대 `bin/com/myserver/main/Main.class` 처럼 파일 경로로 쓰면 안 됨. 패키지 이름으로 불러야 함.

