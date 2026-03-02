
## Creating Jar

애플리케이션 배포 시 수많은 클래스 파일과 디렉토리를 사용자에게 그냥 주면 불편함. 이걸 하나의 파일로 묶기 위해 만듦.

- 클래스 파일(.class)뿐만 아니라 이미지, 사운드 같은 리소스 파일도 포함 가능함.
- ZIP 포맷을 사용해서 파일 용량을 압축함.

CLI 명령어
`jar cvf jarFileName file1 file2 ...`

예시
1. Java 파일 생성 (`HelloJar.java`)
2. 컴파일 (`javac HelloJar.java`)
	- 같은 폴더에 `HelloJar.class` 파일이 생김
3. `jar cvfe MyProgram.jar HelloJar HelloJar.class`
	- jar cvfe (만들Jar 이름) (메인 클래스 이름) (묶을 파일들)
	- c: create, v: verbose, f: file, e: entry point (실행 시작점 지정)
4. `java -jar MyProgram.jar` (실행)


## Manifest

JAR 파일 내부의 `META-INF/MANIFEST.MF` 경로에 위치하는 특수한 파일
JAR 파일의 속성이나 메타데이터를 설명하는 역할

최소한의 내용은 `Manifest-Version: 1.0` 한 줄이지만, 복잡한 설정도 가능
구조
- **Main Section:** 파일의 맨 윗부분. JAR 파일 전체에 적용되는 설정.
- **Individual Sections:** 특정 파일이나 패키지에 적용되는 설정. `Name: 파일명`으로 시작하며, 섹션 간에는 빈 줄로 구분함.

매니페스트 수정/적용 방법
- 직접 `MANIFEST.MF`를 수정하는 게 아니라, 원하는 내용을 담은 텍스트 파일을 따로 만들어서 `jar` 명령어로 합치는 방식
- 옵션 `m`을 사용함.
    - 새로 만들기: `jar cfm [JAR이름] [매니페스트텍스트파일] [포함할파일들...]`
    - 기존 거 업데이트: `jar ufm [기존JAR] [추가할매니페스트내용]`

사용 예시
1. manifest.txt 파일 생성
```
Manifest-Version: 1.0 
Main-Class: HelloJar 
Created-By: MyName
```
2. `jar cfm MyManualJar.jar manifest.txt HelloJar.class`
3. `java -jar MyManualJar.jar`

#### 자세한 설명

JAR 파일 = 많은 클래스, 이미지 등을 압축한 파일
Manifest = 그 파일 내부에 있는 사용설명서

JVM : JAR파일 실행 시 META-INF/MANIFEST.MF 파일을 먼저 본다.

Manifest
1. `Main-Class: HelloJar` : 시작점 설정. (HelloJar.class 안에 있는 main() 실행)
2. 다른 사람들이 만든 라이브러리 (다른 JAR 파일들) 사용할 때
	- Manifest에 `Class-Path: lib/library1.jar lib/library2.jar` 이런 식으로 적으면, 자바가 알아서 그 파일들을 찾아서 연결한다.
3. metadata
	- 파일 버전, 만든 사람, 전자 서명 등등


## Multi-Release JAR Files

Java 9부터 모듈 시스템이 도입되면서 예전에 쓰던 내부 API들이 막히거나 바뀐 경우가 있음.    

라이브러리 개발자 입장에선 Java 8 사용자도 배려해야 하고, Java 9 이상 사용자에게는 최신 기능을 쓰게 해주고 싶은데, 파일 두 개 만들기 싫은 상황이 발생함.

이걸 해결하려고 하나의 JAR 파일 안에 여러 버전의 클래스 파일을 담는 기능 생김.

동작
- **기본(Legacy):** Java 8 이하 버전이 읽을 클래스는 평소처럼 루트에 둠.
    
- **신버전:** Java 9 이상이 읽을 클래스는 `META-INF/versions/버전숫자/` 폴더에 숨겨둠.
    - 예: `META-INF/versions/9/Application.class`
        
- **동작:**
    - Java 8: `META-INF/versions` 폴더를 모르니까 그냥 무시하고 기본 클래스 사용.
    - Java 9+: 자기 버전에 맞는 폴더가 있으면 그걸 우선 사용.
예시
전체 구조
```
src/
├── java8/
│   └── MultiTest.java  (구버전 코드)
└── java9/
    └── MultiTest.java  (신버전 코드)
```

```
# 결과물 저장할 폴더 생성
mkdir -p bin/8 bin/9

# Java 8 버전 컴파일
javac -d bin/8 --release 8 src/java8/MultiTest.java

# Java 9 버전 컴파일
javac -d bin/9 --release 9 src/java9/MultiTest.java
```

multi release jar 생성. 
- `-C bin/8 .`: 기본 베이스는 `bin/8` 폴더 내용을 씀.
- `--release 9 -C bin/9 .`: 버전 9용 내용은 `bin/9` 폴더 내용을 써서 `META-INF/versions/9`에 넣음.
```
jar cf MultiRelease.jar -C bin/8 . --release 9 -C bin/9 .
```

내부 구조
- `jar tf MultiRelease.jar`
```
META-INF/
META-INF/MANIFEST.MF
MultiTest.class                <-- 기본 (Java 8용)
META-INF/versions/9/
META-INF/versions/9/MultiTest.class  <-- 신버전 (Java 9용)
```

*패키지 경로도 모두 동일해야함*







