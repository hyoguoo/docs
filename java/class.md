# 클래스

> 객체를 정의해놓은 것

## 클래스의 구성 요소

```java
class Ogu {
    // 속성(property)
    int height;
    int weight;
    int age;


    // 기능(function)
    void eat() {
        // ...
    }

    void sleep() {
        // ...
    }
}
```

## 인스턴스 생성과 사용

클래스를 선언하는 것은 객체의 설계도만 작성한 것에 불과하므로 해당 클래스를 사용하기 위해서는 인스턴스를 생성해야 한다.  
클래스로부터 객체를 만드는 과정을 클래스의 인스턴스(`instantiate`)라고 하며, 그 객체를 그 클래스의 인스턴스(`instance`)라고 한다.

```java
class Example {

    public void main() {
        Ogu ogu = new Ogu(); // 인스턴스를 참조하기 위한 변수 선언 및 인스턴스 생성
        // 이 시점에는 각 자료형에 해당하는 기본값으로 초기화(ex. int -> 0, String -> null)
        ogu.height = 180;
        ogu.weight = 70;
        ogu.age = 20;
        ogu.eat();
        ogu.sleep();
    }
}
```

## 변수

|   종류    |        선언 위치        |      생성 시기      |                       특징                       |
|:-------:|:-------------------:|:---------------:|:----------------------------------------------:|
| 인스턴스 변수 |       클래스 내부        |   인스턴스 생성됐을 때   |    인스턴스마다 독립적인 저장공간을 가지므로 각 인스턴스마다 다른 값을 가짐    |
| 클래스 변수  |       클래스 내부        | 클래스가 메모리에 올라갈 때 | 모든 인스턴스가 공통된 변수를 공유하며, 인스턴스를 생성하지 않고도 바로 사용 가능 |
|  지역 변수  | 메서드, 생성자, 초기화 블럭 내부 |   메서드가 호출됐을 때   |                 메서드 내에서만 사용가능                  |

```java
class Example {

    int instanceVar; // 인스턴스 변수
    static int classVar; // 클래스 변수

    void method() {
        int localVar; // 지역 변수
    }
}
```

## 메서드

> 특정 작업을 수행하는 일련의 문장들을 하나로 묶은 것

### 메서드의 선언과 구현

```java
class Example {

    // 반환값의 자료형 반환값의 이름(매개변수) { // 메서드 선언 부
    //     // 구현부
    // }
    int add(int a, int b) {
        int result = a + b;
        return result;
    }
}
```

### return

메서드의 반환값이 있는 경우, 메서드의 마지막 문장으로 `return`문을 명시해야 한다.(`void`일 경우 컴파일러에서 자동 추가)  
만약 조건문이 있는 경우에는 항상 반환할 수 있도록 `return`문을 작성해야 한다.

```java
class Example {


    // error
    int add1(int a, int b) {
        if (a > b) {
            return a;
        }
    }

    // ok
    int add2(int a, int b) {
        if (a > b) {
            return a;
        } else {
            return b;
        }
    }
}
```

### static

메서드 중에서 인스턴드와 관계없는(인스턴스 변수나 인스턴스 메서드를 사용자히 않는) 메서드는 클래스 메서드로 선언할 수 있다.

인스턴스가 멤버가 존재하는 시점에는 클래스 멤버가 항상 존재하지만, 클래스 멤버가 존재하는 시점에 인스턴스 멤버가 존재하지 않을 수 있다.  
때문에 같은 클래스 내에 있는 클래스 멤버는 인스턴스 멤버를 사용할 수 없다.  
만약 클래스 멤버 내에서 인스턴스 멤버를 사용해야 할 경우 내부에서 인스턴스를 생성하여 사용할 수 있다.(이 경우 애초에 인스턴스 메서드로 작성해야하는 것은 아닌지 점검 필요)

```java
class Example {

    int instanceVar; // 인스턴스 변수
    static int classVar; // 클래스 변수

    void instanceMethod() {
    }

    static void staticMethod() {
    }

    void instanceMethod2() {
        instanceMethod(); // ok
        staticMethod(); // ok
        System.out.println(instanceVar); // ok
        System.out.println(classVar); // ok
    }

    static void staticMethod2() {
        instanceMethod(); // error
        staticMethod(); // ok
        System.out.println(instanceVar); // error
        System.out.println(classVar); // ok
    }

    static void staticMethod3() {
        Example e = new Example();
        e.instanceMethod(); // ok
        e.staticMethod(); // ok
        System.out.println(e.instanceVar); // ok
        System.out.println(e.classVar); // ok
    }
}
```

## 오버로딩(Overloading)

> 같은 이름의 메서드를 여러 개 정의하는 것

### 오버로딩의 조건

1. 메서드의 이름이 같아야 한다.
2. 매개변수의 개수 또는 타입이 달라야 한다.

```java
class PrintStream {

    void println(int x);

    void println(boolean x);

    void println(double x);

    void println(String x);
    
    // ...
}
```

### 가변인자(varargs)와 오버로딩

가변인자는 메서드의 매개변수의 개수가 일정하지 않을 때 사용한다.

```java
class Example {

    void println(int x) {
        System.out.println(x);
    }

    void println(int x, int y) {
        System.out.println(x + ", " + y);
    }

    void println(int x, int y, int z) {
        System.out.println(x + ", " + y + ", " + z);
    }

    // 가변 인자를 통해 위의 메서드를 대체할 수 있다.
    void println(int... x) {
        for (int i : x) {
            System.out.println(i);
        }
    }
}
```

###### 출처

- Java의 정석