
## Declare

You can define an array variable either as `int[] a;` or as `int a[];` 
Most Java programmers prefer the former style because it neatly separates the type `int[]` (integer array) from the variable name.

`int[] smallPrimes = { 2, 3, 5, 7, 11, 13 };`

```java
String[] authors =  
        {  
                "James Gosling",  
                "Bill Joy",  
                "Guy Steele",  
                // add more names here and put a comma after each name  
        };
```

You can declare an anonymous array: 
- `new int[] { 17, 19, 23, 29, 31, 37 }`

`smallPrimes = new int[] { 17, 19, 23, 29, 31, 37 };`  is shorthand for  
- `int[] anonymous = { 17, 19, 23, 29, 31, 37 };` 
- `smallPrimes = anonymous;`


## accessing

you can fill the elements in an array, for example, by using a loop:
``` java
int[] a = new int[100];  
for (int i = 0; i < 100; i++)  
a[i] = i; // fills the array with numbers 0 to 99
```


초기화되는 기본값

| **배열 타입**    | **예시**                 | **생성 시 초기값** |
| ------------ | ---------------------- | ------------ |
| **숫자 (정수)**  | `int[]`, `long[]`      | **0**        |
| **숫자 (실수)**  | `double[]`             | **0.0**      |
| **불리언**      | `boolean[]`            | **false**    |
| **객체 (참조형)** | `String[]`, `Member[]` | **null**     |

``` java
String[] names = new String[3]; // [null, null, null] 상태  
  
// 에러 발생!  
// names[0]은 현재 null인데, null한테 length()를 물어보니 터짐.  
System.out.println(names[0].length());

for (int i = 0; i < names.length; i++) {  
	names[i] = ""; // 이제 [ "", "", "" ] 상태가 됨  
}
```

length
``` java
//To find the number of elements of an array, use array.length. For example:  
for (int i = 0; i < a.length; i++)  
    System.out.println(a[i]);
```



## for each loop

```java
for (int element : a)    
	System.out.println(element);
```

```java
// i는 자동으로 int로 추론됨
for (var i = 0; i < names.length; i++) {
    System.out.println(names[i]);
}
```

## Array copy

You can copy one array variable into another, but then both variables refer to  
the same array:  
``` java
int[] luckyNumbers = smallPrimes;  
luckyNumbers[5] = 12; // now smallPrimes[5] is also 12
```
변수는 두 개지만, **실제 메모리에 있는 배열은 딱 1개**

If you actually want to copy all values of one array into a new array, use the copyOf method in the Arrays class:
```java
int[] copiedLuckyNumbers = Arrays.copyOf(luckyNumbers, luckyNumbers.length);
//사본을 새로 만듦
luckyNumbers = Arrays.copyOf(luckyNumbers, 2 * luckyNumbers.length);
//복사하면서 배열 크기를 늘리거나 줄일 수 있다.
```


## command line parameter
ava는 JVM 위에서 돌아가기 때문에, JVM이 클래스 이름을 이미 해석하고 **순수하게 사용자가 입력한 인자만** 배열에 담아서 `main`에게 전달

- 명령어: `java MyProgram hello world`
- **배열 상태:**
    - `args[0]`: `"hello"` (첫 번째 실제 인자)
    - `args[1]`: `"world"`

## array sorting

배열 정렬
- **사용법:** `java.util.Arrays` 클래스의 `sort` 메서드를 사용.
``` java
    int[] a = new int[10000];
    // ... 값 채우기 ...
    Arrays.sort(a); // 오름차순 정렬
```   
- **성능:** 내부적으로 튜닝된 **퀵 정렬(QuickSort)**을 사용해서 대부분의 데이터셋에서 매우 효율적임.


## multidimensional arrays

생성
```java
double[][] balances;

// [행의 개수][열의 개수]
balances = new double[NYEARS][NRATES];

// 값 바로 넣어서 생성(단축)
int[][] magicSquare = {
    {16, 3, 2, 13},
    {5, 10, 11, 8},
    {9, 6, 7, 12},
    {4, 15, 14, 1}
};
```

접근
```java
// 1. 행(row)을 하나씩 꺼냄 (row 자체가 double[] 타입)
for (double[] row : balances) {
    // 2. 그 행 안에서 값(value)을 하나씩 꺼냄
    for (double value : row) {
        // 여기서 값 처리
        System.out.println(value);
    }
}
```

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// 1. 바깥쪽 루프 (행, Row)
// matrix.length = 행의 개수 (여기선 3)
for (int i = 0; i < matrix.length; i++) {

    // 2. 안쪽 루프 (열, Column)
    // matrix[i].length = 현재 행의 열 개수 (여기선 3)
    // C와 달리 행마다 길이가 다를 수 있음
    for (int j = 0; j < matrix[i].length; j++) {
        
        System.out.printf("%d ", matrix[i][j]); // C처럼 접근
    }
    System.out.println();
}
```

- **읽기만 할 거다:** 향상된 for 문 (`for (int[] row : matrix)`) 이 적합
- **값을 바꾸거나 인덱스가 필요하다:** `i`, `j` 방식 

## ragged array

`balances[i]`는 숫자 하나가 아니라, $i$번째 행(배열) 전체를 가리키는 **참조(주소)**
`balances[i][j]`는 그 $i$번째 행 배열 안의 $j$번째 값

행(Row) 교체 (Swapping)
``` java
double[] temp = balances[i];
balances[i] = balances[i + 1]; // 주소만 복사됨 (매우 빠름)
balances[i + 1] = temp;
```

ragged array
```java 
final int NMAX = 10;

// 1단계: 전체 행을 담을 '뼈대 배열'만 먼저 만든다.
int[][] odds = new int[NMAX + 1][]; 

// 2단계: 각 행을 돌면서 서로 다른 크기의 배열을 생성해서 끼워 넣는다.
for (int n = 0; n <= NMAX; n++) {
    // n번째 행은 크기가 n+1 (0행은 1개, 1행은 2개 ...)
    odds[n] = new int[n + 1]; 
}

// 3단계: 이제 평소처럼 i, j로 접근 가능
odds[0][0] = 1;
// odds[0][1] 접근 시 에러 발생 (존재하지 않음)

결과 예시:
1
1 1
1 2 1
1 3 3 1
```

