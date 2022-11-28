# 다형성

> 상속과 함께 객체지향개념의 중요한 특징 중 하나로 여러 가지 형태를 가질 수 있는 능력을 의미한다.

자바에서는 조상 클래스 타입의 참조 변수로 자손 클래스의 인스턴스를 참조할 수 있도록 하여 다형성을 프로그램적으로 구현하였다.

```java
class Product {
    int price;
    int bonusPoint;

    Product(int price) {
        this.price = price;
        bonusPoint = (int) (price / 10.0);
    }
}

class Tv extends Product {
    Tv() {
        super(100);
    }

    public String toString() {
        return "Tv";
    }
}

class Example {
    public static void main(String[] args) {
        Product product = new Product(); // Product 클래스의 인스턴스 생성
        Tv tv = new Tv(); // Tv 클래스의 인스턴스 생성
        Product product2 = new Tv(); // Tv 클래스의 인스턴스를 Product 클래스 타입의 참조변수에 저장, Tv 인스턴스의 모든 멤버 사용 불가능
        Tv tv2 = new Product(); // 컴파일 에러
    }
}
```

## instanceof 연산자

> 참조변수가 참조하고 있는 인스턴스의 실제 타입을 알아보기 위해 사용한다.

`instanceof` 연산자의 연산결과는 `boolean` 타입으로 참조변수가 검사한 타입의 인스턴스를 참조하고 있으면 `true`, 그렇지 않으면 `false`를 반환한다.  
`true`를 얻었을 때는 참조변수를 해당 타입으로 형변환 할 수 있다.

```java
class Product {
    int price;
    int bonusPoint;

    Product(int price) {
        this.price = price;
        bonusPoint = (int) (price / 10.0);
    }
}

class Tv extends Product {
    Tv() {
        super(100);
    }

    public String toString() {
        return "Tv";
    }
}

class Example {
    public static void main(String[] args) {
        Product product = new Product(); // Product 클래스의 인스턴스 생성
        Tv tv = new Tv(); // Tv 클래스의 인스턴스 생성
        Product product2 = new Tv(); // Tv 클래스의 인스턴스를 Product 클래스 타입의 참조변수에 저장, Tv 인스턴스의 모든 멤버 사용 불가능

        System.out.println(product instanceof Product); // true
        System.out.println(product instanceof Tv); // false
        System.out.println(tv instanceof Product); // true
        System.out.println(tv instanceof Tv); // true
        System.out.println(product2 instanceof Product); // true
        System.out.println(product2 instanceof Tv); // true
    }
}
```

## 참조변수와 인스턴스 연결

조상 클래스에 선언된 멤버변수와 같은 이름의 인스턴스 변수를 자손 클래스에 중복으로 정의했을 때, 참조변수를 통해 인스턴스 변수에 접근하면 참조변수의 타입에 따라 다른 인스턴스 변수가 사용된다.

```java
class BindingTest {
    public static void main(String[] args) {
        Parent p = new Child();
        Child c = new Child();

        System.out.println("p.x = " + p.x); // 100
        p.method(); // Child method
        System.out.println("c.x = " + c.x); // 200
        c.method(); // Child Method
    }
}

class Parent {
    int x = 100;

    void method() {
        System.out.println("Parent Method");
    }
}

class Child extends Parent {
    int x = 200;

    void method() {
        System.out.println("Child Method");
    }
}
```

멤버 변수가 조상 클래스와 자손 클래스에 중복으로 정의된 경우

- 조상 타입의 참조변수를 사용 : 조상 클래스에 선언된 멤버 변수 사용
- 자손 타입의 참조변수를 사용 : 자손 클래스에 선언된 멤버 변수 사용
- 메서드 사용 : 참조변수의 타입과 관계없이 항상 인스턴스의 타입에 정의된 메서드가 호출

```java
class BindingTest {
    public static void main(String[] args) {
        Parent p = new Child();
        Child c = new Child();

        System.out.println("p.x = " + p.x); // 100
        p.method(); // Parent Method
        System.out.println("c.x = " + c.x); // 100
        c.method(); // Parent Method
    }
}

class Parent {
    int x = 100;

    void method() {
        System.out.println("Parent Method");
    }
}

class Child extends Parent {
}
```

Child 클래스에 아무런 멤버 변수나 메서드가 없는 경우 참조변수의 타입에 관계없이 조상의 멤버들을 사용

```java
class BindingTest {
    public static void main(String[] args) {
        Parent p = new Child();
        Child c = new Child();

        System.out.println("p.x = " + p.x); // 100
        p.method();
        // x=200
        // super.x=100
        // this.x=200
        System.out.println("c.x = " + c.x); // 200
        c.method();
        // x=200
        // super.x=100
        // this.x=200
    }
}

class Parent {
    int x = 100;

    void method() {
        System.out.println("Parent Method");
    }
}

class Child extends Parent {
    int x = 200;

    void method() {
        System.out.println("x=" + x);
        System.out.println("super.x=" + super.x);
        System.out.println("this.x=" + this.x);
    }
}
```

자손 클래스 Child 선언된 인스턴스 변수 x와 조상 클래스 Parent 선언된 인스턴스 변수 x를 구분하기 위해 super.x, this.x를 사용할 수 있다.

## 매개변수의 다형성

매개변수의 다형성은 메서드의 매개변수로 조상 타입의 참조변수를 사용하는 것을 말한다.

```java
class Product {
    int price;
    int bonusPoint;

    Product(int price) {
        this.price = price;
        bonusPoint = (int) (price / 10.0);
    }
}

class Tv extends Product {
}

class Computer extends Product {
}

class Audio extends Product {
}

class Buyer {
    int money = 1000;
    int bonusPoint = 0;
    Product[] item = new Product[10];
    int i = 0;

    void buy(Computer c) {
        this.money -= c.price;
        this.bonusPoint += c.bonusPoint;
    }
    
    void buy(Tv t) {
        this.money -= t.price;
        this.bonusPoint += t.bonusPoint;
    }
    // ...
}
```

위와 같이 구매자가 구매할 수 있는 제품이 여러 종류가 있을 때, 각 제품에 대한 buy 메서드를 구현해야하는데 이를 매개변수의 다형성을 이용하면 다음과 같이 구현할 수 있다.

```java
class Buyer {
    int money = 1000;
    int bonusPoint = 0;
    Product[] item = new Product[10];
    int i = 0;

    void buy(Product p) {
        this.money -= p.price;
        this.bonusPoint += p.bonusPoint;
    }
}
```

### 여러 종류 객체 배열로 다루기

위 예제처럼 `Product` 클래스가 `Tv`, `Computer`, `Audio` 클래스의 조상일 때 `Product` 타입의 참조 변수 배열로 처리할 수 있다.

```java
class Example {
    public static void main(String[] args) {
        Product[] product = new Product[3];
        product[0] = new Tv();
        product[1] = new Computer();
        product[2] = new Audio();
    }
}
```