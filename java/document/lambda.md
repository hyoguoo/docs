# 람다식(Lambda Expression)

자바는 객체지향언어이지만, 자바 8에 도입된 람다식을 통해 함수형 언어의 특징을 사용할 수 있다.  
람다식을 사용하면 메서드를 하나의 식으로 표현할 수 있으며, 코드의 양을 줄이면서 가독성을 높일 수 있다.  
또한 람다식은 메서드의 매개변수로 전달할 수 있으며, 메서드의 결과로 반환할 수도 있다.

## 함수형 인터페이스(Functional Interface)

람다식은 메서드와 비슷한 형태로 작성되지만, 메서드가 아니며 익명 클래스의 객체와 동등하게 표현할 수 있다.  
람다식으로 정의된 익명 객체의 메서드를 호출하기 위해서는 이미 알고 있는 것처럼 참조변수가 있어야 하니 익명 객체 주소를 저장할 참조변수가 필요하다.

```java
class Example {
    public static void main(String[] args) {
        MyFunction f = (int a, int b) -> a > b ? a : b;
    }
}
```

하지만 여기서 참조변수 f의 타입으로 람다식과 일치하는 메서드를 가지고 있는 클래스 또는 인터페이스여야 한다.

```java
interface MyFunction {
    int max(int a, int b);
}
```

이 인터페이스를 구현한 익명 클래스의 객체는 다음과 같이 생성할 수 있다.

```java
class Example {
    public static void main(String[] args) {
        MyFunction f = new MyFunction() {
            public int max(int a, int b) {
                return a > b ? a : b;
            }
        };
        int value = f.max(3, 5);
    }
}
```

위 코드의 메서드 `max()`는 람다식 `(a, b) -> a > b ? a : b`와 동일하기에 라담식으로 대체하면 아래와 같이 된다.

```java
class Example {
    public static void main(String[] args) {
        MyFunction f = (int a, int b) -> a > b ? a : b;
        int value = f.max(3, 5);
    }
}
```

이처럼 `MyFunction` 인터페이스를 구현한 익명 클래스의 객체를 람다식으로 대체할 수 있는 이유는  
람다식이 실제로 익명 객체로 구현되어 있고 `MyFunction` 인터페이스의 `max()`와 람다식의 매배변수의 타입/개수/반환타입이 일치하기 때문이다.  
이처럼 하나의 메서드가 선언된 인터페이스를 정의해 람다식을 다루는 것은 기존 자바의 규칙을 어기지 않는 범위에서 구현이 가능해진다.  
그래서 인터페이스를 통해 람다식을 다루기로 결정되었으며 이를 함수형 인터페이스(`Functional Interface`)라고 한다.

```java

@FunctionalInterface // 함수형 인터페이스라는 것을 명시적으로 알려주는 어노테이션으로 컴파일러 함수형 인터페이스를 올바르게 정의했는지 검사해준다.
interface MyFunction {
    // 여기서 함수형 인터페이스에는 오직 하나의 추상 메서드만 정의되어 있어야 한다.
    int max(int a, int b);

    // default 메서드는 추상 메서드가 아니기 때문에 개수 제약이 없다.
    default void printDefault() {
        System.out.println("Hello");
    }

    // static 메서드는 추상 메서드가 아니기 때문에 개수 제약이 없다.
    static void printStatic() {
        System.out.println("Hello2");
    }
}
```

### 매개변수와 반환 타입

메서드의 매개변수가 `MyFunction` 타입인 경우 해당 메서드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 넘겨주면 된다.

```java
interface MyFunction {
    void myMethod();
}

class Example {
    public static void main(String[] args) {
        MyFunction f = () -> System.out.println("Hello");
        method(f);

        method(() -> System.out.println("Hello")); // 직접 람다식을 매개변수로 지정하는 방법
    }

    static void method(MyFunction f) {
        f.myMethod();
    }
}
```

그리고 반환타입이 `MyFunction` 타입인 경우에도 마찬가지로 람다식을 반환하면 된다.

```java

@FunctionalInterface
interface MyFunction {
    void run();
}

class Example {
    // 매개변수 타입이 MyFunction인 메서드
    public static void execute(MyFunction f) {
        f.run();
    }

    // 반환 타입이 MyFunction인 메서드
    static MyFunction getMyFunction() {
        MyFunction f = () -> System.out.println("Hello");
        return f;
    }

    public static void main(String[] args) {
        // 람다식을 참조변수에 대입
        MyFunction f1 = () -> System.out.println("f1.run");
        // 익명 클래스를 참조변수에 대입
        MyFunction f2 = new MyFunction() {
            public void run() {
                System.out.println("f2.run");
            }
        };
        MyFunction f3 = getMyFunction();

        f1.run();
        f2.run();
        f3.run();

        execute(f1);
        execute(() -> System.out.println("run"));
    }
}
```

## java.util.function 패키지

일반적으로 자주 쓰이는 형식의 메서드를 함수형 인터페이스로 미리 정의해놓은 패키지로 가능하면 미리 정의된 함수형 인터페이스를 사용하는 것이 좋다.

- 자주 쓰이는 함수형 인터페이스

|       인터페이스        |        메서드        | 매개변수 |   반환값   |
|:------------------:|:-----------------:|:----:|:-------:|
| java.lang.Runnable |    void run()     |  없음  |   없음    |
|    Supplier<T>     |      T get()      |  없음  |    T    |
|    Consumer<T>     | void accept(T t)  |  T   |   없음    |
|   Function<T, R>   |   R apply(T t)    |  T   |    R    |
|    Predicate<T>    | boolean test(T t) |  T   | boolean |

이중 `Predicate<T>`는 반환타입이 `boolean`으로 조건식을 람다식으로 표현하는데 사용된다.

```java
class Example {
    public static void main(String[] args) {
        Predicate<String> isEmptyStr = s -> s.length() == 0;
        String s = "";
        if (isEmptyStr.test(s)) {
            System.out.println("s는 빈 문자열입니다.");
        }
    }
}
```

그 외에도 `BiFunction<T, U, R>`과 `BiPredicate<T, U>` 등과 같은 두 개 이상의 매개변수를 가지는 함수형 인터페이스도 있으며,  
`UnaryOperator<T>`와 `BinaryOperator<T>`는 매개변수와 반환값의 타입이 같는 함수형 인터페이스도 존재한다.

## 메서드 참조

메서드 참조는 람다식으로 표현할 수 있는 익명 클래스의 인스턴스를 생성하는 코드를 더 간결하게 표현할 수 있는 방법이다.

```java
class Example {
    public static void main(String[] args) {
        // 람다식
        MyFunction f1 = () -> System.out.println("Hello");
        // 메서드 참조
        MyFunction f2 = System.out::println;
        f1.run();
        f2.run();
    }
}
```

###### 참고자료

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)