## Mutual and Accessor

- `LocalDate aThousandDaysLater = newYearsEve.plusDays(1000);`
The `plusDays` method yields a new LocalDate object, which is then assigned to the `aThousandDaysLater` variable. The original object remains unchanged. We say that the plusDays method does not mutate the object on which it is invoked.

- `GregorianCalendar someDay = new GregorianCalendar(1999, 11, 31);`
Unlike the `LocalDate.plusDays` method, the `GregorianCalendar.add` method is a mutator method. After invoking it, the state of the someDay object has changed.

In contrast, methods that only access objects without modifying them are sometimes called accessor methods. For example, `LocalDate.getYear` and `GregorianCalendar.get` are accessor methods. (also `plusDays`)

**(Mutator):** 메서드 실행 시 객체의 **내부 상태(State)를 변경**
**(Accessor):** 객체의 상태를 변경하지 않고 값을 읽거나, 계산된 **새로운 값을 반환**