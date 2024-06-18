---
layout: editorial
---

# Enums

데이터 타입을 정의해주고 다형성을 구현할 수 있는 기능을 제공한다.  
Java의 enum은 클래스로 정의되어 있어, 단순히 정수형 상수를 정의하는 기능 이상의 기능을 구현할 수 있다.

## Enum 정의와 사용

가장 기본적인 타입 정의와 사용 방법은 아래와 같다.

```java
enum Direction {
    EAST, SOUTH, WEST, NORTH
}

class Unit {

    int x, y;
    Direction direction;

    void moveIf(Direction direction) {
        if (direction.equals(Direction.EAST)) { // enum의 상수는 equals()로 비교 가능
            x++;
        } else if (direction == Direction.SOUTH) { // '==' 연산자를 사용해 equals() 보다 빠른 성능을 기대할 수 있다.
            y--;
        } else if (direction.compareTo(Direction.NORTH)) { // compareTo() 사용 가능
            y++;
        }
//        else if (direction > Direction.WEST) { // enum 에는 비교 연산자인 '<', '>' 사용 불가 
//            x--;
//        }
    }

    void moveSwitch(Direction direction) { // switch 조건식 가능
        switch (direction) {
            case EAST:
                x++;
                break;
            case SOUTH:
                y--;
                break;
            case WEST:
                x--;
                break;
            case NORTH:
                y++;
                break;
        }
    }
}
```

## 열거형과 멤버 변수, 메서드

단순히 상수를 나열하는 것 이상의 기능을 제공하기 위해 클래스와 비슷하게 열거형 상수의 값을 멤버 변수에 저장할 수 있고, 메서드를 정의할 수 있다.

```java
enum Direction {
    EAST(1, ">"), SOUTH(2, "V"), WEST(3, "<"), NORTH(4, "^"); // 열거형 상수를 선언과 동시에 생성자 호출

    private static final Direction[] DIR_ARR = Direction.values();
    private final int value;
    private final String symbol;

    private Direction(int value, String symbol) { // 열거형의 생성자는 private(생략 가능)
        this.value = value;
        this.symbol = symbol;
    }

    public static Direction of(int dir) {
        if (dir < 1 || dir > 4) {
            throw new IllegalArgumentException("Invalid value : " + dir);
        }
        return DIR_ARR[dir - 1];
    }

    public int getValue() {
        return value;
    }

    public String getSymbol() {
        return symbol;
    }

    public Direction rotate(int num) {
        num = (num % 4 + 4) % 4;
        return DIR_ARR[(value - 1 + num) % 4];
    }

    @Override
    public String toString() {
        return name() + " " + getSymbol();
    }
}
```

## Enum과 클래스

enum은 클래스의 일종이기 때문에 클래스에서 할 수 있는 것들을 할 수 있다.  
아래는 enum 내부에 추상 메서드를 선언하고, 각 상수에서 해당 메서드를 구현하는 예시이다.

```java
enum Operation {
    PLUS("+") {
        @Override
        public int eval(int x, int y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public int eval(int x, int y) {
            return x - y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract int eval(int x, int y);

    @Override
    public String toString() {
        return symbol;
    }
}
```

또한, 열거형 상수에는 단순 원시 타입이나 문자열만 저장할 수 있는 것이 아니라 람다식을 저장할 수 있어 아래와 같이 변경할 수 있다.  
기존 메서드를 람다식으로 대체하고 인터페이스를 상속받아 구현한 위와 동일한 기능을 수행한다.

```java
interface Calculator {

    int eval(int x, int y);
}

enum Operation implements Calculator {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y);

    private final String symbol;
    private final IntBinaryOperator op;

    Operation(String symbol, IntBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public int eval(int x, int y) {
        return op.applyAsInt(x, y);
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

## java.lang.Enum 메서드

Java의 모든 열거형은 java.lang.Enum 클래스를 상속받는다. 이 클래스는 열거형을 다루기 위한 여러 유용한 메서드를 제공한다.

|             메서드              |              설명               |
|:----------------------------:|:-----------------------------:|
|         T[] values()         |       모든 열거형 상수를 배열로 반환       |
|        int ordinal()         |        열거형 상수의 순서를 반환         |
|        String name()         |      열거형 상수의 이름을 문자열로 반환      |
| Class<E> getDeclaringClass() | 열거형 상수가 정의된 열거형의 Class 객체를 반환 |

```java
enum Direction {
    EAST(10), SOUTH(20), WEST(30), NORTH(40);

    private final int value;

    Direction(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}

class Test {

    public static void main(String[] args) {
        System.out.println(Direction.EAST.getValue()); // 10
        System.out.println(Arrays.toString(Direction.values())); // [EAST, SOUTH, WEST, NORTH]
        System.out.println(Direction.EAST.ordinal()); // 0
        System.out.println(Direction.EAST.name()); // EAST
        System.out.println(Direction.EAST.getDeclaringClass()); // class Direction
    }
}
```

## 열거형의 내부 구현

```java
enum Direction {
    EAST, SOUTH, WEST, NORTH
}
```

열거형이 위와 같이 정의되어 있을 때 사실은 내부의 상수 하나하나가 `Direction` 클래스의 인스턴스라고 볼 수 있다.  
위의 enum을 클래스로 정의하면 아래와 같이 표현할 수 있다.(동일한 것은 아님)

```java
class Direction {

    public static final Direction EAST = new Direction("EAST");
    public static final Direction SOUTH = new Direction("SOUTH");
    public static final Direction WEST = new Direction("WEST");
    public static final Direction NORTH = new Direction("NORTH");

    private String name;

    private Direction(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)
- [실무 자바 개발을 위한 OOP와 핵심 디자인 패턴](https://school.programmers.co.kr/learn/courses/17778/17778-실무-자바-개발을-위한-oop와-핵심-디자인-패턴)
