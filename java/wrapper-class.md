---
layout: editorial
---

# Wrapper class

> 자바에서는 8개의 기본형을 객체로 다루지 않고 있는데, 이를 객체로 다루기 위해 Wrapper 클래스를 제공한다.

|   기본형   | Wrapper 클래스 |          생성자           |
|:-------:|:-----------:|:----------------------:|
| boolean |   Boolean   | Boolean(boolean value) |
|  char   |  Character  | Character(char value)  |
|  byte   |    Byte     |    Byte(byte value)    |
|  short  |    Short    |   Short(short value)   |
|   int   |   Integer   |   Integer(int value)   |
|  long   |    Long     |    Long(long value)    |
|  float  |    Float    |   Float(float value)   |
| double  |   Double    |  Double(double value)  |

래퍼 클래스에는 `equals()` 메서드가 오버라이딩 되어 있어서, 객체의 내용을 비교할 수 있다.  
또한 `compareTo()` 메서드도 오버라이딩 되어 있어서, 객체의 내용을 비교할 수 있다.

```java
class Example {

    public static void main(String[] args) {
        Integer i1 = new Integer(100);
        Integer i2 = new Integer(100);
        System.out.println(i1 == i2); // false
        System.out.println(i1.equals(i2)); // true
        System.out.println(i1.compareTo(i2)); // 0
    }
}
```

## Boxing

> 기본형을 래퍼 클래스로 변환하는 것을 Boxing, 래퍼 클래스를 기본형으로 변환하는 것을 Unboxing이라고 한다.

```java
class Example {
    public static void main(String[] args) {
        int i = 100;
        Integer i2 = new Integer(i);
        Integer i3 = Integer.valueOf(i);
        Integer i4 = i; // AutoBoxing
        Integer i = new Integer(100);
        int i2 = i.intValue();
        int i3 = i; // AutoUnBoxing
    }
}
```

자바 컴파일러가 자동으로 Boxing, Unboxing을 해주기 때문에, 개발자가 직접 Boxing, Unboxing을 해줄 필요는 없다.  
하지만 Boxing, Unboxing을 하는 경우 추가 연산 작업이 발생하기 때문에, 성능에 영향을 줄 수 있다.

## Number 클래스

> Boolean / Character를 제외한 나머지 Wrapper 클래스들은 Number 클래스를 상속받는다.

Number 클래스에는 `BigInteger`, `BigDecimal` 클래스가 존재하는데, 이는 long, double의 범위를 넘어서는 큰 수를 다룰 때 사용한다.

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
