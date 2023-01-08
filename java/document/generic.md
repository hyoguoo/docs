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

> 메서드의 선언부에 제네릭 타입이 선언된 메서드

```java
class Util {
    public static <T extends Number> int compare(T t1, T t2) {
        double v1 = t1.doubleValue();
        double v2 = t2.doubleValue();
        return Double.compare(v1, v2);
    }
}
```

위의 `makeJuice` 메서드를 제네릭 메서드로 변경하면 다음과 같이 변경된다.

```java
class Juicer {
    // static Juice makeJuice(FruitBox<? extends Fruit> box) {
    static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
        String tmp = "";
        for (Fruit f : box.getList()) tmp += f + " ";
        return new Juice(tmp);
    }
}

public class Main {
    public static void main(String[] args) {
        FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
        FruitBox<Apple> appleBox = new FruitBox<Apple>();

        System.out.println(Juicer.<Fruit>makeJuice(fruitBox));
        System.out.println(Juicer.makeJuice(fruitBox)); // 컴파일러가 타입을 추정할 수 있어 생략 가능
        System.out.println(Juicer.<Apple>makeJuice(appleBox));
        System.out.println(Juicer.makeJuice(appleBox)); // 컴파일러가 타입을 추정할 수 있어 생략 가능
    }
}
```

###### 출처

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)
