
## method parameters

**Java는 무조건 Call by Value**
- 메서드에 변수를 넘길 때, 그 변수의 복사본을 넘겨줌.
- 따라서 메서드 안에서 파라미터를 변경해도 호출한 쪽의 원본 변수는 영향받지 않는다.

- **기본 타입 (Primitive Type - 숫자, boolean 등)**
    - **변경 불가.** 값 자체가 복사되어 넘어가므로, 메서드 안에서 값을 바꿔도 원본은 그대로임.
    - 예: `tripleValue(double x)` 안에서 $x$를 3배로 만들어도 밖의 변수는 안 변함.
        
- **객체 참조 (Object Reference) - 상태 변경**
    - **변경 가능.** 객체의 주소값이 복사되어 넘어감.
    - 복사된 주소(파라미터)를 통해 힙(Heap) 메모리에 있는 실제 객체에 접근해서 데이터를 바꾸는 건 가능함.
    - 예: `harry`의 월급을 인상하는 메서드는 성공함. 
        ``` java
        public static void tripleSalary(Employee x) // works {    
	        x.raiseSalary(200); 
	    }
	    ...
	    harry = new Employee(. . .); 
	    tripleSalary(harry);
        ```
- **객체 참조 (Object Reference) - 객체 교체 (Swap)**
    - **변경 불가.** 이게 핵심임. 파라미터가 가리키는 대상을 다른 객체로 바꿔치기하려 해도 안 됨.
    - 메서드 안의 파라미터($x$, $y$)는 복사본일 뿐이라서, 걔네끼리 서로 바꾸고 메서드가 끝나면 그냥 사라짐. 원본 변수($a$, $b$)는 여전히 원래 객체를 가리킴.
	    ``` java
	    public static void swap(Employee x, Employee y) { 
		    Employee temp = x; 
		    x = y; 
		    y = temp; 
		}
		...
		var a = new Employee("Alice", . . .); 
		var b = new Employee("Bob", . . .); swap(a, b);
		...
		// x refers to Alice, y to Bob 
		Employee temp = x; 
		x = y; 
		y = temp; 
		// now x refers to Bob, y to Alice
	    ```


