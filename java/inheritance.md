---
layout: editorial
---

# 상속

> 기존 클래스를 재사용하여 새로운 클래스를 작성하는 것

상속을 통해 클래스를 작성하면 보다 적은 양의 코드로 새로운 클래스를 작성할 수 있고, 코드를 공통적으로 관리할 수 있어 유지보수에 용이하다.

- 조상 클래스 : 상속을 해주는 클래스(=부모(parent) 클래스, 상위(super) 클래스, 기반(base) 클래스)
- 자손 클래스 : 상속을 받는 클래스(=자식(child) 클래스, 하위(sub) 클래스, 파생(derived) 클래스)

```java
class Parent {
    private String name;
    private int age;
    // Child 클래스의 address 미존재 
}

class Child extends Parent {
    // Parent 클래스의 name, age 필드를 상속받음
    private String address;
}

class Child extends Parent1, Parent2 { // error
    // ...
}

class Parent extends Object { // Object 클래스는 모든 클래스의 조상
    // ...
}
```

- 자바에서는 관계 명확성 및 신뢰성을 위해 단일 상속만 허용한다.
- 모든 클래스는 `java.lang.Object` 클래스를 상속받는다.(생략 가능)

## 상속 이외에 클래스를 재사용하는 방법

포함(Composition)을 통해 클래스를 재사용할 수 있다.

```java
class Circle {
    private int x;
    private int y;
    private int radius;
}
```

```java
class Circle {
    private Point point = new Point();
    private int radius;
}

class Rectangle {
    private Point point = new Point();
    private int width;
    private int height;
}

class Point {
    private int x;
    private int y;
}
```

### 포함 vs 상속

- 원(Circle)은 점(Point)이다. (is-a 관계) -> 상속
- 원(Circle)은 점(Point)을 포함한다. (has-a 관계) -> 포함

이 경우 원(Circle)은 점(Point)을 포함하므로 상속보다는 포함이 적합하다.

## 오버라이딩(Overriding)

> 조상 클래스로부터 상속받은 메서드의 내용을 변경하는 것

```java
class Point {
    int x;
    int y;

    String getLocation() {
        return "x : " + x + ", y : " + y;
    }
}

class Point3D extends Point {
    int z;

    String getLocation() { // Overriding
        return "x : " + x + ", y : " + y + ", z : " + z;
    }
}
```

오버라이딩의 조건

- 조상 클래스의 메서드와 이름이 같아야 한다.
- 매개변수의 수와 타입이 모두 같아야 한다.
- 반환 타입이 같아야 한다.
- 접근 제어자는 조상 클래스의 메서드보다 좁은 범위로 변경할 수 없다.(public > protected > default > private)
- 조상 클래스의 메서드보다 더 많은 수의 예외를 선언할 수 없다.(조상 클래스의 메서드가 예외를 선언하지 않은 경우, 자손 클래스에서 선언할 수 없다.)
- 인스턴스 메서드를 static 메서드로 또는 그 반대로 변경할 수 없다.

### 오버로딩(Overloading) vs 오버라이딩(Overriding)

- 오버로딩(Overloading) : 같은 이름의 메서드를 정의하는 것
- 오버라이딩(Overriding) : 상속받은 메서드의 내용을 변경하는 것

```java
class Point {
    int x;
    int y;

    String getLocation() {
        return "x : " + x + ", y : " + y;
    }
}

class Point3D extends Point {
    int z;

    String getLocation() { // Overriding
        return "x : " + x + ", y : " + y + ", z : " + z;
    }

    String getLocation(int x, int y, int z) { // Overloading
        return "x : " + x + ", y : " + y + ", z : " + z;
    }
}
```

## super

> 조상 클래스의 멤버와 자손 클래스의 멤버의 이름이 같을 때 둘을 구별하기 위해 사용하는 참조변수

```java
class Point {
    int x;
    int y;

    String getLocation() {
        return "x : " + x + ", y : " + y;
    }
}

class Point3D extends Point {
    int z;

    String getLocation() { // Overriding
        return "x : " + x + ", y : " + y + ", z : " + z;
    }

    String getSuperLocation() {
        return super.getLocation();
    }
}
```

### super()

> 조상 클래스의 생성자

자손 클래스의 인스턴스를 생성하면, 자손 멤버와 조상의 멤버가 합쳐진 하나의 인스턴스가 생성된다.  
자손 클래스의 생성자는 조상 클래스의 생성자를 호출하여 조상 클래스의 멤버들이 초기화되도록 해야 한다.  
첫 줄에서 조상클래스 생성자를 호출애햐하는 이유는 조상 클래스의 멤버들이 자손 클래스의 멤버보다 먼저 초기화되어야 하기 때문이다.  
마찬가지로 조상 클래스의 생성자도 상속 관계를 거슬러 올라가면서 최종적으론 Object 클래스의 생성자까지 호출하개 된다.  
첫 줄에서 조상클래스 생성자를 호출하지 않으면 컴파일러가 자동으로 super()를 호출한다.

```java
class Point {
    int x;
    int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

class Point3D extends Point {
    int z;

    Point3D(int x, int y, int z) {
        // super(); // 컴파일러가 자동으로 추가 -> 정의 된 생성자가 없어 오류 발생 
        this.x = x;
        this.y = y;
        this.z = z;
    }
}
```

위와 같이 조상 클래스의 생성자를 호출하지 않으면 컴파일러가 자동으로 super()를 호출하지만 정의 된 생성자가 없어 오류가 발생하므로 조상 클래스에 정의 된 생성자를 호출해야 한다.

```java
class Point /* extends Object */ {
    int x;
    int y;

    Point(int x, int y) {
        // super(); // 컴파일러가 자동으로 추가 -> 조상인 Object 클래스 생성자 Object() 호출
        this.x = x;
        this.y = y;
    }

    String getLocation() {
        return "x : " + x + ", y : " + y;
    }
}

class Point3D extends Point {
    int z;

    Point3D(int x, int y, int z) {
        super(x, y); // 조상 클래스의 생성자 Point(int x, int y) 호출
        this.z = z;
    }

    String getLocation() { // Overriding
        return "x : " + x + ", y : " + y + ", z : " + z;
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)