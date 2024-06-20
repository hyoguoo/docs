---
layout: editorial
---

# Object class

`java.lang` 패키지는 자바프로그래밍에 가장 기본이 되는 클래스로. `import` 없이 사용할 수 있다.  
모든 클래스의 최고 조상으로, 때문에 모든 클래스는 Object 클래스의 멤버를 모두 사용할 수 있다.  
아래 11개의 메서드를 가지고 있으며 멤버 변수는 없다.

|         Object class method         |                         description                         |
|:-----------------------------------:|:-----------------------------------------------------------:|
|     `protected Object clone()`      |                            객체 복제                            |
| `public boolean equals(Object obj)` |                       객체의 내용이 같은지 비교                        |
|     `protected void finalize()`     |                 객체 소멸 전에 호출(가비지 컬렉터에 의해 호출)                 |
|      `public Class getClass()`      |               객체의 클래스 정보를 담고 있는 Class 인스턴스 반환               |
|       `public int hashCode()`       |                        객체의 해시코드를 반환                         |
|     `public String toString()`      |                         객체를 문자열로 반환                         |
|       `public void notify()`        |                객체 자신을 사용하려고 기다리는 쓰레드를 하나만 깨움                |
|      `public void notifyAll()`      |                객체 자신을 사용하려고 기다리는 쓰레드를 모두 깨움                 |
|        `public void wait()`         | 다른 쓰레드가 `notify()` 혹은 `notifyAll()`을 호출할 때까지 지정된 시간동안 대기 지정 |

이중 `equals()`, `hashCode()`, `toString()` 메서드는 자주 사용된다.

## equals(Object obj)

Object 클래스의 `equals()` 메서드는 두 객체의 주소값(=동일성 비교)을 비교한다.  
컴퓨터 메모리 특성 상 두 개의 객체가 같은 주소값을 갖는 일은 없지만, 두 개 이상의 참조변수가 같은 주소값을 갖는 것은 가능하다.

```java
class Point {

    int x;
    int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

public class EqualsTest {

    public static void main(String[] args) {
        Point a = new Point(2, 3);
        Point b = new Point(2, 3);
        Point c = a; // a와 c는 같은 객체를 참조

        // 동일성 비교(비교 대상이 같은 인스턴스인지)
        System.out.println(a == b); // false
        System.out.println(a == c); // true

        // 동등성 비교(비교 대상이 같은 값을 갖는지), 하지만 Point 클래스는 equals() 메서드를 오버라이딩하지 않았기 때문에 동일성 비교와 같다.
        System.out.println(a.equals(b)); // false
        System.out.println(a.equals(c)); // true
    }
}
```

Point 클래스는 `equals()` 메서드를 오버라이딩하지 않았기 때문에, `Object` 클래스의 메서드가 호출하게 되면서 두 객체의 주소값(=동일성 비교)을 비교한다.  
만약 두 객체의 멤버 변수 값을 비교(=동등성 비교)하고 싶다면, `equals()` 메서드를 오버라이딩하여 사용할 수 있다.

```java
class Point {

    int x;
    int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }
}

public class EqualsTest {

    public static void main(String[] args) {
        Point a = new Point(2, 3);
        Point b = new Point(2, 3);
        Point c = a;

        // 동일성 비교(비교 대상이 같은 인스턴스인지)
        System.out.println(a == b); // false
        System.out.println(a == c); // true

        // 동등성 비교(비교 대상이 같은 값을 갖는지)
        System.out.println(a.equals(b)); // true
        System.out.println(a.equals(c)); // true
    }
}
```

### [String 클래스의 equals](string-class#String-클래스의-equals)

## hashCode()

해싱기법에 사용되는 `hash function`을 구현한 것으로 찾고자 하는 값을 입력하면, 그 값이 저장된 주소(`hash code`)를 반환한다.  
String 클래스의 `hashCode()` 메서드는 문자열의 내용을 이용하여 해시코드를 생성한다.

```java
public class HashCodeTest {

    public static void main(String[] args) {
        String s1 = "ogu";
        String s2 = "ogu";
        System.out.println(s1.hashCode()); // 109981
        System.out.println(s2.hashCode()); // 109981
        Object o1 = new Object();
        Object o2 = new Object();
        System.out.println(o1.hashCode()); // 424058530
        System.out.println(o2.hashCode()); // 321001045
    }
}
```

### hashCode() & equals()

동일한 객체는 동일한 메모리 주소를 갖는다는 것을 의미하므로, 동일한 객체는 동일한 해시코드를 가져야한다.  
특히 Hash를 이용한 자료구조를 이용할 때, 객체의 해시코드를 이용하여 저장하므로, `equals()`,`hashCode()` 두 메서드는 같이 오버라이딩해야 한다.  
아래 예시처럼 `equals()`를 오버라이딩하여 의도대로 비교해 주었지만, `hashCode()`를 오버라이딩하지 않아서 의도하지 않은 문제가 발생할 수 있다.

```java
class Animal {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null)
            return false;
        if (obj == this)
            return true;
        if (obj.getClass() != this.getClass())
            return false;
        Animal animal = (Animal) obj;
        return this.name.equals(animal.getName());
    }
}

class Example {

    public static void main(String[] args) {
        Animal animal1 = new Animal();
        Animal animal2 = new Animal();
        animal1.setName("animal");
        animal2.setName("animal");

        System.out.println(animal1.equals(animal2)); // true
        System.out.println(animal1.hashCode()); // 321001045
        System.out.println(animal2.hashCode()); // 791452441

        Set<Animal> animals = new HashSet<>();
        animals.add(animal1);
        animals.add(animal2);
        System.out.println(animals); // [Animal@2f2c9b19, Animal@13221655]
        // hashCode()가 일치하지 않아서, 같은 객체로 인식하지 않아 중복 저장된다.
    }
}
```

`hashCode()`를 오버라이딩하여 `name`을 이용하여 해시코드를 생성해주면, 같은 객체로 인식하여 중복 저장되지 않는다.

```java
class Example {

    public static void main(String[] args) {
        Animal animal1 = new Animal();
        Animal animal2 = new Animal();

        Set<Animal> animals = new HashSet<>();
        animals.add(animal1);
        animals.add(animal2);
        System.out.println(animals); // [Animal@1ad9d]
    }
}

class Animal {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null)
            return false;
        if (obj == this)
            return true;
        if (obj.getClass() != this.getClass())
            return false;
        Animal animal = (Animal) obj;
        return this.name.equals(animal.getName());
    }

    @Override
    public int hashCode() {
        return this.name.hashCode();
    }
}
```

## toString()

`toString()` 메서드는 객체의 정보를 문자열로 반환하며, 일반적으로 인스턴스나 클래스에 대한 정보 또는 인스턴스 변수의 값을 문자열로 반환한다.  
보통 로그를 남길 때, `toString()` 메서드를 오버라이딩하여 사용한다.

```java
class Example {

    public static void main(String[] args) {
        Animal animal = new Animal();
        animal.setName("ogu");
        System.out.println(
                animal.toString()); // Animal{name='ogu'}, print 메서드를 호출하면 자동으로 toString() 메서드가 호출된다.
    }
}

class Animal {
    // ...

    @Override
    public String toString() {
        return "Animal{" + "name='" + name + '\'' + '}';
    }
}
```

## clone()

`clone()` 메서드는 객체를 복제하여 새로운 객체를 생성한다.  
Object 클래스의 `clone()` 메서드는 인스턴스 변수의 값만 복사하기 때문에, 인스턴스 변수가 참조형일 경우, 참조형 변수의 값 아닌 참조값이 복사된다.  
따라서, `clone()` 메서드를 오버라이딩하여 참조형 변수의 값도 복사해주어야 한다.

```java
class Example {

    public static void main(String[] args) {
        Animal animal = new Animal();
        animal.setName("ogu");
        animal.addList(5);
        Animal clone = animal.clone();
        System.out.println(clone.equals(animal)); // true
        System.out.println(clone.getName()); // ogu
        System.out.println(clone.getList()); // [5]
        clone.addList(9);
        System.out.println(clone.getList()); // [5, 9]
        System.out.println(animal.getList()); // [5, 9]
    }
}

class Animal implements Cloneable {

    private String name;
    private List<Integer> list = new ArrayList<>();

    public List<Integer> getList() {
        return list;
    }

    public void addList(int number) {
        list.add(number);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null)
            return false;
        if (obj == this)
            return true;
        if (obj.getClass() != this.getClass())
            return false;
        Animal animal = (Animal) obj;
        return this.name.equals(animal.name) && this.list.equals(animal.list);
    }

    @Override
    public int hashCode() {
        return this.name.hashCode();
    }

    @Override
    public Animal clone() {
        try {
            Animal clone = (Animal) super.clone();
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

새로운 List 값을 생성하여 복사해주면, 참조형 변수의 값도 복사하면서 문제를 해결할 수 있다.

```java
class Example {

    // ...
    @Override
    public Animal clone() {
        try {
            Animal clone = (Animal) super.clone();
            clone.list = new ArrayList<>(this.list);
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

## getClass()

`getClass()` 메서드는 객체의 클래스 정보를 반환한다.

```java
class Example {

    public static void main(String[] args) {
        Animal animal = new Animal();
        System.out.println(animal.getClass()); // class Animal
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)
- [실무 자바 개발을 위한 OOP와 핵심 디자인 패턴](https://school.programmers.co.kr/learn/courses/17778/17778-실무-자바-개발을-위한-oop와-핵심-디자인-패턴)
