---
layout: editorial
---

# Generics

> 컴파일 시에 타입을 체크하는 기능

```java
// 제네릭 클래스(= 제네릭 타입)
class Box<T> {

    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}

class Main {

    public static void main(String[] args) {
        Box<String> box = new Box<String>();
        box.set("Hello");
        String str = box.get();
    }
}
```

위처럼 선언된 클래스를 제네릭 클래스(인터페이스)라고 하며, 둘을 통틀어 제네릭 타입(Generic Type)이라고 한다.

## 제네릭에 사용되는 용어

- 제네릭 타입에 사용되는 용어

|                Terminology                |              Example               |
|:-----------------------------------------:|:----------------------------------:|
|       Parameterized type(매개변수화 타입)        |           `List<String>`           |
|     Actual type parameter(실제 타입 매개변수)     |              `String`              |
|           Generic type(제네릭 타입)            |             `List<E>`              |
|     Formal type parameter(정규 타입 매개변수)     |                `E`                 |
|  Unbounded wildcard type(비한정적 와일드카드 타입)   |             `List<?>`              |
|              Raw type(로 타입)               |               `List`               |
|    Bounded type parameter(한정적 타입 매개변수)    |        `<E extends Number>`        |
|      Recursive type bound(재귀적 타입 한정)      |    `<T extends Comparable<T>>`     |
|    Bounded wildcard type(한정적 와일드카드 타입)    |      `List<? extends Number>`      |
|          Generic method(제네릭 메서드)          | `static <E> List<E> asList(E[] a)` |
| type token(타입 토큰) or type literal(타입 리터럴) |           `String.class`           |

- 제네릭에 사용되는 형식 타입 변수(강제 사항 X)

| Type  | Description |
|:-----:|:-----------:|
| `<T>` |    Type     |
| `<E>` |   Element   |
| `<K>` |     Key     |
| `<V>` |    Value    |
| `<N>` |   Number    |

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

그리고 제네릭 배열 타입의 참조변수를 선언하는 것은 가능하지만 제네릭 타입의 배열을 생성하는 것은 불가능하다.

```java
class Box<T> {

    T[] itemArr; // 가능

    T[] toArray() { // 불가능
        return new T[10]; // 컴파일 에러
    }
}
```

new 연산자는 컴파일 시점에 타입 T가 정확히 알아야 하는데, 생성 시점에는 타입 T가 무엇인지 알 수 없기 때문에 불가능하다.  
만약 배열을 생성하고 싶다면 아래와 같이 복사를 한 뒤 형변환 하여 사용하는 방법이 있다.

```java
class Box<T> {

    T[] itemArr;

    T[] toArray() {
        return (T[]) Arrays.copyOf(itemArr, itemArr.length);
    }
}
```

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
        Box<Apple> Box = new Box<>(); // 타입 추론에 의해 생략 가능
        Box<Apple> grapeBox = new Box<Grape>(); // 컴파일 에러

    }
}
```

- 제네릭 클래스의 인스턴스를 생성할 때는 타입 변수에 대입할 실제 타입을 지정해야 한다.
- 참조 변수와 생성자에 대입된 타입이 일치해야하며, 일치하지 않으면 컴파일 에러가 발생한다.
- 타입을 생략하면 컴파일러가 코드에서 타입 정보를 추론하고 자동으로 설정해준다.(= 타입 추론)

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
        for (Fruit f : box.getList())
            tmp += f + " ";
        return new Juice(tmp);
    }

    // 제네릭 타입이 다르더라도 오버로딩이 성립하지 않아 컴파일 에러
    static Juice makeJuice(FruitBox<Apple> box) {
        String tmp = "";
        for (Fruit f : box.getList())
            tmp += f + " ";
        return new Juice(tmp);
    }
}
```

이를 해결하기 위해 오버로딩을 사용하면 제네릭 타입이 다른 것만으로는 오버로딩이 성립하지 않아 메서드 중복 정의가 되어 컴파일 에러가 발생한다.  
대신 아래와 같이 와일드 카드를 사용해 해결할 수 있다.

```java
class Fruit {
    // Fruit 클래스의 내용은 생략
}

class Apple extends Fruit {
    // Apple 클래스의 내용은 생략
}

class FruitBox<T extends Fruit> {

    private final List<T> list = new ArrayList<>();

    public void add(T fruit) {
        list.add(fruit);
    }

    public List<T> getList() {
        return list;
    }
}

class Juice {

    private final String content;

    public Juice(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Juice(" + content + ")";
    }
}

class Juicer {

    static Juice makeJuice(FruitBox<? extends Fruit> box) {
        StringBuilder tmp = new StringBuilder();
        for (Fruit f : box.getList()) {
            tmp.append(f).append(" ");
        }
        return new Juice(tmp.toString());
    }
}

class Main {

    public static void main(String[] args) {
        FruitBox<Fruit> fruitBox = new FruitBox<>();
        FruitBox<Apple> appleBox = new FruitBox<>();

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
        System.out.println(
                "a E Type : " + a.get().getClass().getName()
        ); // a E Type : java.lang.String

        System.out.println("b data : " + b.get()); // b data : 10
        System.out.println(
                "b E Type : " + b.get().getClass().getName()
        ); // b E Type : java.lang.Integer

        // 제네릭 메소드 Integer
        System.out.println(
                "<T> returnType : " + a.<Integer>genericMethod(3).getClass()
                        .getName()
        ); // <T> returnType : java.lang.Integer
        System.out.println(
                "<T> returnType : " + a.genericMethod(3).getClass()
                        .getName()
        ); // <T> returnType : java.lang.Integer, 타입 추론에 의해 생략 가능
        // 제네릭 메소드 String
        System.out.println(
                "<T> returnType : " + a.<String>genericMethod("ABCD").getClass()
                        .getName()
        ); // <T> returnType : java.lang.String
        System.out.println(
                "<T> returnType : " + a.genericMethod("ABCD").getClass()
                        .getName()
        ); // <T> returnType : java.lang.String, 타입 추론에 의해 생략 가능
        // 제네릭 메소드 ClassName b
        System.out.println(
                "<T> returnType : " + a.<ClassName>genericMethod(b).getClass()
                        .getName()
        ); // <T> returnType : ClassName
        System.out.println(
                "<T> returnType : " + a.genericMethod(b).getClass()
                        .getName()
        ); // <T> returnType : ClassName, 타입 추론에 의해 생략 가능
    }
}
```

위와 같이 구현을 하게되면 클래스에서 지정한 제네릭 타입과 별도로 메서드에서 지정한 제네릭 타입을 사용할 수 있다.  
위 예시에선 넘겨받은 인자를 통해 타입을 추론할 수 있기 때문에 제네릭 타입을 생략할 수 있다.

### static

제네릭 메서드 선언은 정적 메서드로 선언할 때 사용할 수 있는데, 클래스에 선언된 제네릭 타입과는 별도로 독립적인 제네릭 타입을 사용해야 한다.

```java
class ClassName<E> {

    // 클래스의 제네릭 타입 E를 사용
    E genericMethod0(E o) {
        return o;
    }

    // 에러 발생, static 메서드는 객체가 생성되기 이전 시점에 메모리에 올라가기 때문에 클래스에 선언 된 E 유형을 사용할 수 없음
    static E genericStaticMethod1(E o) {
        return o;
    }

    // 같은 타입 변수 E를 사용하지만 독립적인 타입 E 사용
    static <E> E genericStaticMethod2(E o) {
        return o;
    }

    // 제네릭 클래스의 E 타입과 다른 독립적인 타입 T 사용 가능
    static <T> T genericStaticMethod3(T o) {
        return o;
    }

    // 에러 발생, 반환 타입에 클래스 제네릭 타입 E를 사용할 수 없음
    static <T> E genericStaticMethod4(T o) {
        return o;
    }
}
```

## Generic Type Erasure

제네릭 타입은 컴파일 시에만 유효하고, 컴파일 후에는 런타임 중에는 타입 정보가 사라지게 된다.  
`extends`를 통한 제한이 없는 경우에는 `Object`로 대체되고, `extends`를 통해 제한이 있는 경우에는 제한된 타입으로 대체된다.

```java
// 컴파일 전
class Test<T> {

    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}

// 컴파일 후
class Test {

    private Object t;

    public void set(Object t) {
        this.t = t;
    }

    public Object get() {
        return t;
    }
}
```

```java
// 컴파일 전
class Test<T extends Number> {

    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}

// 컴파일 후
class Test {

    private Number t;

    public void set(Number t) {
        this.t = t;
    }

    public Number get() {
        return t;
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)
- [Stranger's LAB Tistory](https://st-lab.tistory.com/153)
