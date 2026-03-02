
## if

```java
if (yourSales >= 2 * target)
{
performance = "Excellent";
bonus = 1000;
}
else if (yourSales >= 1.5 * target)
{
performance = "Fine";
bonus = 500;
}
else if (yourSales >= target)
{
performance = "Satisfactory";
bonus = 100;
}
else
{
System.out.println("You're fired");
}
```


## while

```java
while (balance < goal)
{
balance += payment;
double interest = balance * interestRate / 100;
balance += interest;
years++;
}
System.out.println(years + " years.");
```


## for
```java
for (int i = 10; i > 0; i--)    
	System.out.println("Counting down . . . " + i); 
System.out.println("Blastoff!");
```


## switch

switch expression (java 14+)
```java
Scanner in = new Scanner(System.in);  
System.out.print("Select an option (1, 2, 3, 4) ");  
int choice = in.nextInt();  
switch (choice)  
        {  
        case 1 ->  
        . . .  
        case 2 ->  
        . . .  
        case 3 ->  
        . . .  
        case 4 ->  
        . . .  
default ->  
        System.out.println("Bad input");  
}
```

```java
String input = . . .;  
        switch (input.toLowerCase())  
        {  
        case "yes", "y" ->  
        . . .  
        case "no", "n" ->  
        . . .  
		default ->  
        . . .
}
```

전통 switch 문
```java
int choice = . . .;  
        switch (choice)  
        {  
        case 1:  
        . . .  
        break;  
        case 2:  
        . . .  
        break;  
        case 3:  
        . . .  
        break;  
        case 4:  
        . . .  
        break;  
		default:  
        // bad input  
        . . .  
        break;  
}
```

## labeled break

```java
read_data:
while (조건) {
    ...
}
```
`read_data` → **라벨 이름**
라벨은 **`break` 또는 `continue`의 대상**

```java
Scanner in = new Scanner(System.in);  
int n;  
read_data:  
        while (. . .) // this loop statement is tagged with the label  
        {  
        . . .  
            for (. . .) // this inner loop is not labeled  
            {     
                System.out.print("Enter a number >= 0: ");  
                n = in.nextInt();  
                if (n < 0) // should never happen—can't go on  
                    break read_data;                
                    // break out of read_data loop                  
                . . .  
            }  
        }  
        // this statement is executed immediately after the labeled break    
        if (n < 0) // check for bad situation  
        {  
        // deal with bad situation  
        }  
        else  
        {  
        // carry out normal processing  
        }
```

- `break read_data` : 지금 실행 중인 모든 반복문을 무시하고  `read_data:`가 붙은 while문 바깥으로 즉시 이동하라
- `continue`도 라벨 가능 