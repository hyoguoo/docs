---
layout: editorial
---

# 인터페이스

> 일종의 추상클래스로 추상클래스보다 추상화 정도가 높아 추상메서드와 상수만을 멤버로 가질 수 있다.

## 인터페이스의 정의

키워드로 class 대신 interface를 사용하며, 접근 제어자에 제한이 있으며 생략할 수 있다.

- 멤버변수 : `public static final` 제어자를 가져야 한다.
- 메서드 : `public abstract` 제어자를 가져야 한다.

```java
interface 인터페이스명 {
    public static final type name = value;

    public abstract type method();
}
```

## 인터페이스의 구현

인터페이스 그 자체로는 인스턴스를 생성할 수 없으며, 인터페이스를 구현한 클래스의 인스턴스를 생성해야 한다.  
인터페이스를 구현하는 클래스는 인터페이스에 선언된 모든 메서드를 구현해야 하며 `implements` 키워드를 사용한다.(상속과 동시에 할 수 있다.)

```java
class Fighter extends Unit implements Fightable {
    public void move() {
        System.out.println("move");
    }

    public void attack() {
        System.out.println("attack");
    }
}
```

만약 인터페이스를 구현한 클래스가 인터페이스에 선언된 모든 메서드를 구현하지 않으면, 해당 클래스는 추상클래스가 되어야 한다.

```java
abstract class Fighter extends Unit implements Fightable {
    public void move() {
        System.out.println("move");
    }
}
```

인터페이스에서 `public abstract`로 지정되있고, 오버라이딩 할 때는 조상의 메서드보다 접근 제어자를 좁힐 수 없다,  
떄문에 클래스에서 구현을 할 때 접근 제어자를 `public`으로 지정해야한다.

## 인터페이스의 상속

인터페이스는 인터페이스로부터만 상속 받을 수 있으며, 클래스와 달리 다중 상속이 가능하다.

```java
interface Movable {
    void move();
}

interface Attackable {
    void attack();
}

interface Fightable extends Movable, Attackable {
}
```

## 인터페이스를 이용한 다형성 구현

리턴타입이 인터페이스인 경우, 인터페이스를 구현한 클래스의 인스턴스를 리턴할 수 있다.

```java
interface Parseable {
    public abstract void parse(String fileName);
}

class ParserManager {
    public static Parseable getParser(String type) {
        if (type.equals("XML")) {
            return new XMLParser();
        } else {
            Parseable p = new HTMLParser();
            return p;
        }
    }
}

class XMLParser implements Parseable {
    public void parse(String fileName) {
        System.out.println(fileName + "- XML parsing completed.");
    }
}

class HTMLParser implements Parseable {
    public void parse(String fileName) {
        System.out.println(fileName + "- HTML parsing completed.");
    }
}

class ParserTest {
    public static void main(String[] args) {
        Parseable parser = ParserManager.getParser("XML"); // == Parseable parser = new XMLParser();
        parser.parse("document.xml"); // document.xml- XML parsing completed.
        System.out.println(parser.getClass().getName()); // XMLParser
        parser = ParserManager.getParser("HTML"); // == Parseable parser = new HTMLParser();
        parser.parse("document2.html"); // document2.html- HTML parsing completed.
        System.out.println(parser.getClass().getName()); // HTMLParser
    }
}
```

## 인터페이스의 활용

- 클래스를 사용하는 쪽(User)과 클래스를 제공하는 쪽(Provider)이 있다.
- 클래스를 사용하는 쪽에서는 자신이 필요로 하는 메서드만 호출하면 되며, 내용은 신경 쓰지 않아도 된다.

```java
class A {
    void methodA(B b) {
        b.methodB();
    }
}

class B {
    void methodB() {
        System.out.println("methodB");
    }
}

class InterfaceTest {
    public static void main(String[] args) {
        A a = new A();
        a.methodA(new B());
    }
}
```

위와 같이 클래스 A가 클래스 B의 메서드를 호출하는 경우, 클래스 B가 이미 작성되어 있어야 한다.  
그리고 클래스 B의 메서드의 선언부가 변경되면 클래스 A도 변경되어야 한다.(높은 결합도)
인터페이스를 사용하면 클래스 B의 내용을 신경 쓰지 않고, 클래스 A는 클래스 B의 메서드를 호출하는 것만 알면 된다.

```java

class A {
    void methodA(I i) {
        i.methodB();
    }
}

interface I {
    public abstract void methodB();
}

class B implements I {
    public void methodB() {
        System.out.println("methodB");
    }
}

class InterfaceTest {
    public static void main(String[] args) {
        A a = new A();
        a.methodA(new B());
    }
}
```

결국 클래스 A는 클래스 B의 메서드를 호출하지만 클래스 B와 직접적인 관계가 없고 인터페이스 I를 통해 호출하게 되며, 인터페이스 I의 영향만 받게 된다.

```java
class A {
    void methodA() {
        I i = InstanceManager.getInstance();
        i.methodB();
        System.out.println(i.toString());
    }
}

interface I {
    public abstract void methodB();
}

class B implements I {
    public void methodB() {
        System.out.println("methodB");
    }

    public String toString() {
        return "class B";
    }
}

class InterfaceTest {
    public static void main(String[] args) {
        A a = new A();
        a.methodA();
    }
}
```

인스턴스를 직접 생성하지 않고, `InstanceManager`를 통해 인스턴스를 얻어서 제공받을 수 있는데,  
이렇게 되면 클래스 A는 클래스 B의 내용을 신경 쓰지 않고, 클래스 B의 인스턴스를 얻어서 사용할 수 있으며 다른 클래스의 인스턴스로 변경해도 `InstanceManager`만 변경하면 된다.

## static / default 메서드

기존에는 `public abstract`만 선언할 수 있었지만 JDK 1.8부터 `default`와 `static` 메서드를 선언할 수 있다.

```java
interface Calculator {
    static int execMulti(int a, int b) {
        return a * b;
    }

    int plus(int a, int b);

    int multi(int a, int b);

    default int execPlus(int a, int b) {
        return a + b;
    }
}

class CalculatorImpl implements Calculator {

    @Override
    public int plus(int pre, int post) {
        return pre + post;
    }

    @Override
    public int multi(int pre, int post) {
        return pre * post;
    }

    // default 메서드를 오버라이딩 하지 않으면 기본적으로 인터페이스의 default 메서드를 사용
//    @Override
//    public int execPlus(int a, int b) {
//        return a + b;
//    }
}

class CalculatorTest {
    public static void main(String[] args) {
        Calculator cal = new CalculatorImpl();

        System.out.println(cal.plus(3, 9)); // 12
        System.out.println(cal.multi(3, 9)); // 27

        System.out.println(cal.execPlus(3, 9)); // 12
        System.out.println(cal.execMulti(3, 9)); // error
        System.out.println(Calculator.execMulti(3, 9)); // 27
    }
}
```

|  method  | 내부 구현 가능 여부 | 재정의 여부 |    호출 방법    |
|:--------:|:-----------:|:------:|:-----------:|
|  static  |     가능      |  불가능   | 인터페이스명.메서드명 |
| default  |     가능      |   가능   |  클래스명.메서드명  |
| abstract |     불가능     |   가능   |  클래스명.메서드명  |

## 익명 구현 객체(Anonymous Implement Object)

- 구현 클래스를 정의하지 않고, 인터페이스를 구현하는 객체를 생성하는 방법
- 인터페이스를 구현하는 익명 구현 객체는 인터페이스를 구현하는 클래스의 인스턴스를 생성하는 것과 같다.

```java
interface Calculator {
    int exec(int a, int b);
}

class CalculatorTest {
    public static void main(String[] args) {
        Calculator cal = new Calculator() {
            @Override
            public int exec(int a, int b) {
                return a + b;
            }
        };

        System.out.println(cal.exec(3, 9)); // 12
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)