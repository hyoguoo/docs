# Generics

> 컴파일 시에 타입을 체크하는 기능

```java
class Box<T> {
    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}
```

- Box<T> : 제네릭 클래스
- T : 제네릭 타입 파라미터
    - T를 타입 변수(`type variable`)라고 하며, 다른 것을 사용해도 된다.(E, K, V, N, S, U, T, R 등)
- Box : 원시 타입(raw type)

## 제네릭의 제한

모든 객체에 대해 동일하게 동작해야하는 `static` 메서드나 `static` 필드에서는 타입 변수를 사용할 수 없다.  
`static` 멤버는 인스턴스 생성과 상관없이 사용할 수 있기 때문에, 대입 된 타입의 종류에 관계없이 동일해야하기 때문이다.

```java
class Box<T> {
    static T t; // 컴파일 에러

    static int compare(T t1, T t2) { // 컴파일 에러
        //...
    }
}
```

그리고 제네릭 배열 타입의 참조변수를 선언하는 것은 가능하지만 제네릭 타입의 배열을 생성하는 것도 불가능하다.

```java
class Box<T> {
    T[] itemArr; // 가능

    T[] toArray() { // 불가능
        return new T[10]; // 컴파일 에러
    }
}
```

new 연산자는 컴파일 시점에 타입 T가 정확히 알아야하지만 제네릭 타입의 배열을 생성할 때는 타입 T가 정확히 알 수 없기 때문이다.  
만약 제네릭 배열을 생성해야한다면 `Reflection API`의 `newInstance()`와 같은 동적으로 객체를 생성하는 메서드로 배열을 생성하거나  
Object 배열을 생성해서 복사한 다음에 `T[]`로 형변환하면 된다.

## 제네릭 클래스 생성과 사용

```java
class Box<T> {
    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<Apple> appleBox = new Box<Apple>();
        Box<Apple> Box = new Box<>();
        Box<Apple> grapeBox = new Box<Grape>(); // 컴파일 에러

    }
}
```

- 제네릭 클래스의 인스턴스를 생성할 때는 타입 변수에 대입할 실제 타입을 지정해야 한다.
- 참조 변수와 생성자에 대입된 타입이 일치해야하며 생략 가능하다.

## 제한된 제네릭 클래스

제네릭 클래스의 타입 변수에는 어떤 타입이라도 대입될 수 있지만, `extends`와 `&`를 사용해 특정 타입만 대입되도록 제한할 수 있다.

```java
interface Eatable {
}

class Box<T extends Fruit & Eatable> {
    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}

class Fruit implements Eatable {
    public String toString() {
        return "Fruit";
    }
}

class Apple extends Fruit {
    public String toString() {
        return "Apple";
    }
}

class Toy {
    public String toString() {
        return "Toy";
    }
}

public class Main {
    public static void main(String[] args) {
        Box<Fruit> fruitBox = new Box<Fruit>();
        Box<Apple> appleBox = new Box<Apple>();
        Box<Toy> toyBox = new Box<Toy>(); // 컴파일 에러

        fruitBox.set(new Fruit());
        fruitBox.set(new Apple());
        appleBox.set(new Apple());
        toyBox.set(new Toy()); // 컴파일 에러
    }
}
```

## 와일드 카드

```java
class Juicer {
    static Juice makeJuice(FruitBox<Fruit> box) {
        String tmp = "";

        for (Fruit f : box.getList()) {
            tmp += f + " ";
        }

        return new Juice(tmp);
    }
}
```

위와 같은 특정 제네릭을 받는 메서드가 있을 경우 `FruitBox<Apple>`과 같이 `Fruit`의 자손 클래스를 대입할 수 없다.

```java
class Juicer {
    static Juice makeJuice(FruitBox<Fruit> box) {
        String tmp = "";
        for (Fruit f : box.getList()) tmp += f + " ";
        return new Juice(tmp);
    }

    // 컴파일 에러
    static Juice makeJuice(FruitBox<Apple> box) {
        String tmp = "";
        for (Fruit f : box.getList()) tmp += f + " ";
        return new Juice(tmp);
    }
}
```

이를 해결하기 위해 오버로딩을 사용하면 제네릭 타입이 다른 것만으로는 오버로딩이 성립하지 않아 메서드 중복 정의가 되어 컴파일 에러가 발생한다.  
대신 와일드 카드를 사용해 해결할 수 있다.

```java
class FruitBox<T extends Fruit> extends Box<T> {
}

class Juicer {
    static Juice makeJuice(FruitBox<? extends Fruit> box) {
        String tmp = "";
        for (Fruit f : box.getList()) tmp += f + " ";
        return new Juice(tmp);
    }
}

public class Main {
    public static void main(String[] args) {
        FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
        FruitBox<Apple> appleBox = new FruitBox<Apple>();

        fruitBox.add(new Fruit());
        fruitBox.add(new Apple());
        appleBox.add(new Apple());
        appleBox.add(new Apple());

        System.out.println(Juicer.makeJuice(fruitBox));
        System.out.println(Juicer.makeJuice(appleBox));
    }
}
```

- `<? extends T>` : T와 그 자손들만 가능
- `<? super T>` : T와 그 조상들만 가능
- `<?>` : 모든 타입이 가능(=<? extends Object>)

## 제네릭 메서드

> 메서드의 선언부에 제네릭 타입이 선언된 메서드, 반환 타입 이전에 제네릭 타입을 선언

인스턴스 생성할 때의 타입과 상관없이 독립적으로 제네릭 유형을 선언하여 쓸 수 있다.

```java
class ClassName<E> {

    private E element;

    // 제네릭 파라미터 메소드
    void set(E element) {
        this.element = element;
    }

    // 제네릭 타입 반환 메소드
    E get() {
        return element;
    }

    // 제네릭 메소드
    <T> T genericMethod(T o) {
        return o;
    }
}

public class Main {
    public static void main(String[] args) {

        ClassName<String> a = new ClassName<String>();
        ClassName<Integer> b = new ClassName<Integer>();

        a.set("10");
        b.set(10);

        System.out.println("a data : " + a.get()); // a data : 10
        System.out.println("a E Type : " + a.get().getClass().getName()); // a E Type : java.lang.String

        System.out.println("b data : " + b.get()); // b data : 10
        System.out.println("b E Type : " + b.get().getClass().getName()); // b E Type : java.lang.Integer

        // 제네릭 메소드 Integer
        System.out.println("<T> returnType : " + a.genericMethod(3).getClass().getName()); // <T> returnType : java.lang.Integer
        // 제네릭 메소드 String
        System.out.println("<T> returnType : " + a.genericMethod("ABCD").getClass().getName()); // <T> returnType : java.lang.String
        // 제네릭 메소드 ClassName b
        System.out.println("<T> returnType : " + a.genericMethod(b).getClass().getName()); // <T> returnType : ClassName
    }
}
```

위와 같이 구현을 하게되면 클래스에서 지정한 제네릭 타입과 별도로 메서드에서 지정한 제네릭 타입을 사용할 수 있다.

### static

위와 같이 제네릭 메서드 선언은 정적 메서드로 선언할 때 사용할 수 있다.  
제네릭 타입은 해당 클래스 객체가 인스턴스화 했을 때 외부에서 지정된 타입으로 결정되는데, `static`은 프로그램 실행 시 메모리에 이미 올라가 있기 때문에 클래스와 별도로 독립적인 제네릭을 사용해야 한다.

```java
class ClassName<E> {
    // 에러 발생, static 메서드는 객체가 생성되기 이전 시점에 메모리에 올라가기 때문에 E 유형을 가져올 수 없다.
    static E genericStaticMethod1(E o) {
        return o;
    }

    // 제네릭 클래스의 E 타입과 다른 독립적인 타입 
    static <E> E genericStaticMethod2(E o) {
        return o;
    }

    static <T> T genericStaticMethod3(T o) {
        return o;
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(ClassName.<String>genericStaticMethod2("ABCD")); // ABCD
        System.out.println(ClassName.<Integer>genericStaticMethod2(10)); // 10

        System.out.println(ClassName.<String>genericStaticMethod3("ABCD")); // ABCD
        System.out.println(ClassName.<Integer>genericStaticMethod3(10)); // 10
    }
}
```

###### 출처

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)
- [제네릭(Generic)의 이해](https://st-lab.tistory.com/153)