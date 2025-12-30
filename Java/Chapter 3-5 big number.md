
- `BigInteger`: 임의의 정밀도를 가진 정수 연산 (자릿수 제한 없는 정수).
- `BigDecimal`: 임의의 정밀도를 가진 부동소수점 연산 (금융 계산 등 정확한 소수점 처리가 필요할 때).

```java
import java.math.BigInteger;  
  
public class BigNumber {  
    public static void main(String[] args) {  
        BigInteger result = BigInteger.ONE;  
  
        for (int i=1; i<=100; i++){  
            BigInteger value = BigInteger.valueOf(i); 
            //일반 int/long 을 BigInteger로 변환  
            result = result.multiply(value); //연산자 대신 method        
        }  
  
        System.out.print("100 팩토리얼 결과: ");  
        System.out.println(result);  
  
        BigInteger bigNum = new BigInteger("2222322446294204455297398934619099
        67206666939096499990979600");  
        System.out.println("문자열로 만든 큰 수: "+ bigNum);  
    }  
}
```

``` java
import java.math.BigDecimal;  
  
public class BigDecimalTrap {  
    public static void main(String[] args) throws Exception{  
  
        // 1.double 값을 그대로 생성자에 넣음  
        //부동소수점(`double`) 자체의 근사값 표현 방식 때문에 예측 불가능한 오차가 발생함.
        BigDecimal unsafe = new BigDecimal(0.1);  
  
        // 2.String으로 값을 넣음  
        BigDecimal safe = new BigDecimal("0.1");  
  
        System.out.println("=== 0.1을 저장했을 때 차이 ===");  
        System.out.println("double 생성자: " + unsafe);  
        // 0.100000000000000005551115... (오차)  
  
        System.out.println("String 생성자: " + safe);  
        //0.1  
        if (unsafe.equals(safe)) {  
            System.out.println("equal");  
        } else {  
            System.out.println("not equal");  
        }  
    }  
}
```


Java는 C++과 달리 **연산자 오버로딩(Operator Overloading)**을 지원하지 않음. 따라서 `+`, `*` 같은 산술 연산자 대신 메서드를 사용

