---
layout: editorial
---

# Class(클래스)

> 객체를 생성하기 위한 템플릿으로 객체 지향 프로그래밍에서 중요한 개념

## 클래스의 구성 요소

```java
class Ogu {

    // 속성(property)
    int height;
    int weight;
    int age;

    // 기능(function), 메서드(method)
    void eat() {
        // ...
    }

    void sleep() {
        // ...
    }
}
```

- 속성(property): 클래스 내부에 선언된 변수로, 클래스 객체마다 각각의 값이 가질 수 있는 특징이나 상태를 나타냄
- 메서드(method): 클래스 내부에 선언된 함수로, 객체의 상태를 조작하거나 객체 간의 상호작용을 처리하는데 사용

## 인스턴스 생성과 사용

클래스를 선언하는 것은 객체의 설계도만 작성한 것에 불과하므로 해당 클래스를 사용하기 위해서는 인스턴스를 생성해야 한다.  
클래스로부터 객체를 만드는 과정을 클래스의 인스턴스화(`instantiate`)라고 하며, 그 객체를 그 클래스의 인스턴스(`instance`)라고 한다.

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

```java
class Example {

    int add(int a, int b) { // 메서드 선언 부
        // 메서드 구현 부
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

메서드 중에서 인스턴스와 관계없는(인스턴스 변수나 인스턴스 메서드를 사용하지 않는) 메서드는 클래스 메서드로 선언할 수 있다.  
인스턴스 메서드나 멤버는 사용할 수 없는데, 그 이유는 아래와 같다.

- 클래스 멤버가 존재하는 시점에는 인스턴스 멤버가 존재하지 않을 수 있음
- 각 인스턴스마다 독립적인 저장공간을 가지므로 클래스 멤버가 인스턴스 멤버를 참조할 수 없음

만약 클래스 멤버 내에서 인스턴스가 필요한 경우엔 내부에서 인스턴스를 생성하여 사용할 수 있다.(이 경우 애초에 인스턴스 메서드로 작성해야하는 것은 아닌지 점검 필요)

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

같은 이름의 메서드를 여러 개 정의하는 것으로, 매개변수의 개수나 타입을 다르게 하여 같은 이름으로 메서드를 여러 개 정의할 수 있다.

```java
class PrintStream {

    void println(int x);

    void println(boolean x);

    void println(double x);

    void println(String x);

    void println(int x, int y) {
        System.out.println(x + ", " + y);
    }

    void println(int x, int y, int z) {
        System.out.println(x + ", " + y + ", " + z);
    }

    // 가변 인자를 통해 매개변수의 개수가 일정하지 않을 때 사용할 수 있다.
    void println(int... x) {
        for (int i : x) {
            System.out.println(i);
        }
    }
```

## 생성자(Constructor)

인스턴스가 생성될 때 호출되는 인스턴스 초기화 메서드로, 클래스의 이름과 동일하며 리턴 타입이 없다.

### 생성자 실행 과정

1. 연산자 `new`에 의해 힙(heap) 영역에 인스턴스가 생성된다.
2. 생성자가 호출되어 인스턴스 변수들이 초기화된다.
3. 연산자 `new`의 결과로 생성된 인스턴스의 주소가 반환되어 참조변수에 저장된다.

### 생성자

모든 클래스에는 반드시 하나 이상의 생성자가 존재해야하며, 정의하지 않은 경우에는 컴파일러가 `기본 생성자`를 추가한다.  
매개변수가 있는 생성자를 만들 수 있으며, 생성자를 하나라도 정의하면 컴파일러는 기본 생성자를 추가하지 않는다.

```java
class Example {

    int x;

    Example(int x) {
        this.x = x;
    }
}

class Main {

    public static void main(String[] args) {
        Example e = new Example(); // 컴파일 에러
        Example e = new Example(10); // ok
    }
}
```

## 변수 초기화

> 변수를 선언하고 처음으로 값을 저장하는 것

1. 명시적 초기화 : 변수를 선언과 동시에 초기화하는 것
2. 초기화 블럭 : 여러 문장으로 이루어진 초기화 코드를 블럭으로 묶어 놓은 것
    1. 인스턴스 초기화 블럭 : 인스턴스 변수를 초기화하는 블럭
    2. 클래스 초기화 블럭 : 클래스 변수를 초기화하는 블럭
3. 생성자 : 인스턴스가 생성될 때 호출되는 인스턴스 초기화 메서드

```java
class InitBlock {

    static int classVar = 10; // 클래스 변수의 명시적 초기화
    int instanceVar = 10; // 인스턴스 변수의 명시적 초기화

    // 클래스 초기화 블록을 이용한 초기화
    static {
        classVar = 20;
    }

    // 인스턴스 초기화 블록을 이용한 초기화
    {
        instanceVar = 20;
    }

    // 생성자를 이용한 초기화
    InitBlock() {
        instanceVar = 30;
    }
}
```

### 멤버변수 초기화 시기와 순서

1. 클래스 변수 : 클래스가 처음 로딩될 때 단 한번 초기화
    - 기본값 -> 명시적 초기화 -> 클래스 초기화 블럭
2. 인스턴스 변수 : 인스턴스가 생성될 때 마다 각 인스턴스별로 초기화
    - 기본값 -> 명시적 초기화 -> 인스턴스 초기화 블럭 -> 생성자

```java
class InitTest {

    static int cv = 1;
    int iv = 1;

    static {
        cv = 2;
    }

    {
        iv = 2;
    }

    public InitTest() {
        iv = 3;
    }
}
```

|         -         | cv | iv |
|:-----------------:|:--:|:--:|
|   (클래스 초기화)기본값    | 0  | -  |
| (클래스 초기화)명시적 초기화  | 1  | -  |
|    클래스 초기화 블럭     | 2  | -  |
|   (인스턴스 초기화)기본값   | 2  | 0  |
| (인스턴스 초기화)명시적 초기화 | 2  | 1  |
|    인스턴스 초기화 블럭    | 2  | 2  |
|        생성자        | 2  | 3  |

1. `cv`가 메모리(method area)에 생성되고, 타입 기본 값인 0이 저장
2. 명시적 초기화(`int cv = 1`)가 수행되어 1로 저장
3. 클래스 초기화 블럭이 수행되어 2로 저장
4. InitTest 클래스 인스턴스가 생성되면서 `iv`가 메모리(heap)에 생성되고, 타입 기본 값인 0이 저장
5. 명시적 초기화(`int iv = 1`)가 수행되어 1로 저장
6. 인스턴스 초기화 블럭이 수행되어 2로 저장
7. 생성자가 수행되어 3으로 저장

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)
