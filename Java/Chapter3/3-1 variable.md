The keyword public is called an access modifier; these modifiers control the level of access other parts of a program have to this code.

## Basic structure:
``` java
public class ClassName 
{   
	public static void main(String[] args)   
	{     
		program statements   
	} 
}
```

## variable
Starting with Java 10, you do not need to declare the types of local variables if they can be inferred from the initial value. Simply use the keyword var instead of the type: 
``` java
var vacationDays = 12; // vacationDays is an int 
var greeting = "Hello"; // greeting is a String
```

## constant
In Java, you use the keyword final to denote a constant. 
``` java
 public class Constants {    
	 public static void main(String[] args)    {       
		 final double CM_PER_INCH = 2.54;       
		 double paperWidth = 8.5;       
		 double paperHeight = 11;       
		 System.out.println("Paper size in centimeters: " + paperWidth * CM_PER_INCH + " by " + paperHeight * CM_PER_INCH);    
	} 
}

```

The keyword **final** indicates that you can assign to the variable once, and then its value is set once and for all. It is customary to name constants in all uppercase.
It is probably more common in Java to create a constant so it’s available to multiple methods inside a single class. These are usually called class constants.

Set up a class constant with the keywords static final. Here is an example of using a class constant:
``` java
public class Constants2 {     
	public static final double CM_PER_INCH = 2.54;     
	public static void main(String[] args)     {       
		double paperWidth = 8.5;       
		double paperHeight = 11;       
		System.out.println("Paper size in centimeters: "          + paperWidth * CM_PER_INCH + " by " + paperHeight * CM_PER_INCH);     
	} 
} 
```
Note that the definition of the class constant appears outside the main method. Thus, the constant can also be used in other methods of the same class. Furthermore, if the constant is declared, as in this example, public, methods of other classes can also use it—in our example, as `Constants2.CM_PER_INCH.`


## enum
``` java
enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }; //define

Size s = Size.MEDIUM; //declare
```

A variable of type Size can hold only one of the values listed in the type declaration, or the special value null that indicates that the variable is not set to any value at all.


## math
Consider the expression n % 2. Everyone knows that this is 0 if n is even and 1 if n is odd. Except, of course, when n is odd and negative. Then it is -1.

Consider this problem. You compute the position of the hour hand of a clock. An adjustment is applied, and you want to normalize to a number between 0 and 11. That is easy: 
- (position + adjustment) % 12. 
But what if the adjustment is negative? Then you might get a negative number. So you have to introduce a branch, or use 
- ((position + adjustment) % 12 + 12) % 12.

The floorMod method makes it easier: 
- `floorMod(position + adjustment, 12)` 
always yields a value between 0 and 11. (Unfortunately, floorMod gives negative results for negative divisors, but that situation doesn’t often occur in practice.)


## conversion
![[Pasted image 20251229144213.png]]
- legal conversions
When two values are combined with a binary operator (such as n + f where n is an integer and f is a floating-point value), both operands are converted to a common type before the operation is carried out. 
-  If either of the operands is of type double, the other one will be converted to a double. 
- Otherwise, if either of the operands is of type float, the other one will be converted to a float. 
- Otherwise, if either of the operands is of type long, the other one will be converted to a long. 
- Otherwise, both operands will be converted to an int.

## cast
``` java
double x = 9.997; 
int nx = (int) x;
```

## switch expression

``` java
String seasonName = switch (seasonCode)  
{  
    case 0 -> "Spring";  
    case 1 -> "Summer";  
    case 2 -> "Fall";  
    case 3 -> "Winter";  
    default -> "???";  
};
```

You can provide multiple labels for each case, separated by commas:  
```java
int numLetters = switch (seasonName)  
{  
    case "Spring", "Summer", "Winter" -> 6;  
    case "Fall" -> 4;  
    default -> -1;  
}; 
```
 
When you use the switch expression with enumerated constants, you need  
not supply the name of the enumeration in each label—it is deduced from  
the  
switch value. For example:  
```java
enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }; . . .  
Size itemSize = . . .;  
String label = switch (itemSize)  
{  
    case SMALL -> "S"; // no need to use Size.SMALL  
    case MEDIUM -> "M";  
    case LARGE -> "L";  
    case EXTRA_LARGE -> "XL";  
};
```
  
In the example, it was legal to omit the default since there was a case for  
each possible value.

