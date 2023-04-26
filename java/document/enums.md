# Enums

> 관련된 상수를 편리하게 선언하기 위한 것

## Enum 정의와 사용

```java
enum Direction {
    EAST, SOUTH, WEST, NORTH
}

class Unit {
    int x, y;
    Direction direction;

    void moveIf(Direction direction) {
        if (direction == Direction.EAST) { // '==' 연산자를 사용해 equals() 보다 빠른 성능을 기대할 수 있다.
            x++;
        } else if (direction > Direction.WEST) { // '<', '>' 사용 불가
            y--;
        } else if (direction.compareTo(Direction.WEST)) { // compareTo() 사용 가능
            x--;
        }
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

## java.lang.Enum 메서드

|             메서드              |              설명               |
|:----------------------------:|:-----------------------------:|
|         T[] values()         |       모든 열거형 상수를 배열로 반환       |
|        int ordinal()         |        열거형 상수의 순서를 반환         |
|        String name()         |      열거형 상수의 이름을 문자열로 반환      |
| Class<E> getDeclaringClass() | 열거형 상수가 정의된 열거형의 Class 객체를 반환 |

## 열거형에 멤버 추가

```java
enum Direction {
    EAST(1, ">"), SOUTH(2, "V"), WEST(3, "<"), NORTH(4, "^"); // 열거형 상수를 모두 정의 후에 멤버 추가 가능

    private static final Direction[] DIR_ARR = Direction.values();
    private final int value;
    private final String symbol;

    private Direction(int value, String symbol) { // 열거형의 생성자는 private으로 생략 가능 
        this.value = value;
        this.symbol = symbol;
    }

    public int getValue() {
        return value;
    }

    public String getSymbol() {
        return symbol;
    }

    public static Direction of(int dir) {
        if (dir < 1 || dir > 4) {
            throw new IllegalArgumentException("Invalid value : " + dir);
        }
        return DIR_ARR[dir - 1];
    }

    public Direction rotate(int num) {
        num = num % 4;

        if (num < 0) {
            num += 4;
        }

        return DIR_ARR[(value - 1 + num) % 4];
    }

    @Override
    public String toString() {
        return name() + getSymbol();
    }
}
```

## 열거형의 이해

```java
enum Direction {
    EAST, SOUTH, WEST, NORTH
}
```

열거형이 위와 같이 정의되어 있을 때 내부의 상수 하나하나가 `Direction` 객체이다.  
클래스로 정의하게 된다면 아래와 같이 정의할 수 있다.

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
}
```

Direction 클래스의 static final 필드인 EAST, SOUTH, WEST, NORTH 값은 객체의 주소이며, 바뀌지 않는 값이므로 `==`로 비교가 가능하다.

###### 참고자료

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)