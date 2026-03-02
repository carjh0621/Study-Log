
## Javadoc

JDK에 포함된 도구로, 소스 파일에서 HTML 문서를 생성해줌.
소스 코드와 문서를 한 파일(`/** ... */`)에서 관리할 수 있음

- **형식:** `/**`로 시작하고 `*/`로 끝냄.
    
- **구조:** 자유로운 텍스트 설명(Free-form text) + 태그(`@`로 시작하는 것들).
    
- **요약:** 첫 문장은 전체 요약으로 간주해서 Javadoc이 자동으로 요약 페이지에 추출함.

- **HTML 사용:** `<em>`, `<strong>`, `<ul>`, `<img>` 같은 HTML 태그 사용 가능.
    
- **코드 표시:** `{@code ...}`를 사용하면 부등호(`<`) 같은 거 이스케이프 안 하고 편하게 코드 작성 가능.
    
- **이미지 첨부:** 소스 디렉토리에 `doc-files`라는 하위 폴더 만들고 이미지 넣은 뒤 링크 걸면 됨.

```
# src 폴더에 있는 모든 자바 파일의 문서를 docs 폴더에 생성
javadoc -d docs -sourcepath src -subpackages . -encoding UTF-8 -charset UTF-8 -docencoding UTF-8

# 하나만
javadoc -d docs -encoding UTF-8 -charset UTF-8 -docencoding UTF-8 src/Calculator.java

# 기본 브라우저
explorer.exe docs/index.html
```


## Comments

### Class 
- `import` 문 **다음에**, 그리고 `class` 정의 **바로 앞에** 위치해야 함.
- 순서: `package` 선언 -> `import` 문 -> **[클래스 주석]** -> `public class ...`
```java
package com.example.game; // 1. 패키지
import java.util.*; // 2. 임포트

/**
 * 트럼프 카드를 나타내는 클래스임.
 * 무늬(Suit)와 숫자(Value)를 가짐.
 * (여기가 클래스 주석 위치)
 */
public class Card { // 3. 클래스 정의
    // ...
}
```

### Method

- 설명하려는 메서드 **바로 윗줄**에 작성해야 함.
**주요 태그 (Tags)**\
- **`@param 변수명 설명`**
    - 매개변수(파라미터) 설명.
    - 설명이 길면 여러 줄 써도 되고, HTML 태그도 사용 가능.
    - 파라미터가 여러 개면 `@param` 태그끼리 뭉쳐서 써야 함.
        
- **`@return 설명`**
    - 반환값(리턴값) 설명.
    - `void` 메서드면 생략함.
        
- **`@throws 클래스명 설명`**
    - 메서드 실행 중 발생할 수 있는 예외(에러) 설명.
```java
/**
 * 직원의 연봉을 인상함.
 *
 * @param byPercent 인상할 비율 (예: 10은 10%를 의미함)
 * @return 인상된 급여 금액
 * @throws IllegalArgumentException 비율이 음수일 경우 발생
 */
public double raiseSalary(double byPercent) {
    if (byPercent < 0) throw new IllegalArgumentException();
    double raise = salary * byPercent / 100;
    salary += raise;
    return raise;
}
```

### Fields

- Public 필드만 작성하면 됨.
- 일반적으로 `public static final` 같은 상수가 주석 작성 대상

### General Comments

**1. 메타 데이터 태그**
- **`@since 텍스트`**: 이 기능이 도입된 버전 표기. (예: `@since 1.7.1`)
- **`@author 이름`**: 작성자 표기. 여러 명이면 태그 여러 개 쓰면 됨. 
- **`@version 텍스트`**: 현재 버전 표기.
    
**2. 링크/참조 태그 (`@see`)**
- 문서 하단의 See Also (참고) 섹션에 링크를 추가함.
- 코드 참조 : `패키지.클래스#멤버 라벨`

    - **주의:** 클래스와 멤버(메서드/변수) 사이는 `.`이 아니라 `#`을 써야 함.  
    - `@see com.horstmann.corejava.Employee#raiseSalary(double)`
    - 패키지나 클래스명은 생략 가능(현재 위치 기준).
        
- (웹 링크): `<a href="...">...</a>` 형태의 URL.
- (텍스트): `"..."` 큰따옴표로 감싼 텍스트. (예: 책 이름)
    

**3. 인라인 링크 태그 (`{@link}`)**
- **`{@link 패키지.클래스#멤버 라벨}`**
- `@see`와 문법은 같은데, 이건 주석 설명 문장 중간에 하이퍼링크를 넣을 때 씀.
    
**4. 검색 태그 (`{@index}`)**
- Java 9부터 추가됨.
- **`{@index 키워드}`** 형태로 쓰면 생성된 문서 검색창에서 해당 키워드가 검색됨.


### Package

패키지 전체 설명은 각 패키지 폴더마다 별도의 파일을 만들어야 함.

- 방법 1: `package-info.java` 파일 생성
    - 폴더 안에 `package-info.java`라는 이름의 자바 파일을 만듦.    
    - 내용은 `/** ... */` 주석을 먼저 쓰고, 그 뒤에 `package 패키지명;` 선언만 넣음.
    - 다른 코드는 넣으면 안 됨.
        
- 방법 2: `package.html` 파일 생성 
    - 폴더 안에 `package.html` 파일을 만듦.
    - `<body>`와 `</body>` 태그 사이에 있는 내용만 Javadoc이 추출해서 문서로 만듦.

```
/**
 * Core Java 책의 예제 코드들을 포함하는 패키지임.
 * <p>
 * 이 패키지는 직원 관리 및 급여 계산과 관련된 클래스들을 포함함.
 * </p>
 *
 * @author Cay Horstmann
 * @since 1.0
 */
package com.horstmann.corejava;
```


## Comment Extraction

소스 파일이 있는 폴더가 아니라, 패키지 폴더를 포함하고 있는 상위 폴더에서 실행

- 패키지 지정: `javadoc -d [결과폴더명] [패키지명]`
- 여러 패키지: `javadoc -d [결과폴더명] [패키지1] [패키지2] ...`
- 패키지 없음(파일만): `javadoc -d [결과폴더명] *.java`

Fine-tuning

- **`-author`, `-version`**: 기본적으로는 `@author`랑 `@version` 태그가 문서에 안 나옴. 나오게 하려면 이 옵션을 켜야 함.
- **`-link [URL]`**: 표준 라이브러리(String, List 등)를 클릭했을 때 Oracle 공식 문서로 연결해줌. 
    - 예: `-link http://docs.oracle.com/javase/9/docs/api`
- **`-linksource`**: 문서에서 메서드나 클래스를 클릭하면, 실제 소스 코드(HTML로 변환된 것)를 보여줌.


overview comment
- 모든 소스 파일을 아우르는 메인 대문 페이지 설명
	-  `overview.html` 파일을 만들고 `<body>...</body>` 사이에 내용을 적음.
	- 실행: 명령어에 `-overview overview.html` 옵션을 추가함.
	- 결과: 생성된 문서 내비게이션 바에서 "Overview"를 누르면 이 내용이 나옴.

예시
```
javadoc -d docs \
    -sourcepath src \
    -author -version \
    -link https://docs.oracle.com/en/java/javase/21/docs/api \
    -linksource \
    -overview src/overview.html \
    com.horstmann.corejava
```
- `docs` 폴더에 저장.
    
- 작성자, 버전 표시.
    
- Java 21 공식 문서랑 링크 연결.
    
- 소스 코드 보기 기능 추가.
    
- `overview.html`을 대문으로 사용.