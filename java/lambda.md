---
layout: editorial
---

# 람다식(Lambda Expression)

자바는 객체지향언어이지만, 자바 8에 도입된 람다식을 통해 함수형 언어의 특징을 사용할 수 있다.  
람다식을 사용하면 메서드를 하나의 식으로 표현할 수 있으며, 코드의 양을 줄이면서 가독성을 높일 수 있다.  
또한 람다식은 메서드의 매개변수로 전달할 수 있으며, 메서드의 결과로 반환할 수도 있다.

## 함수형 인터페이스(Functional Interface)

함수형 인터페이스는 단 하나의 추상 메서드를 가지는 인터페이스를 말한다.  
일반적으로 인터페이스를 구현한 익명 클래스의 객체는 다음과 같이 생성할 수 있다.

```java
interface MyFunction {
    int max(int a, int b);
}


class Example {
    public static void main(String[] args) {
        MyFunction f = new MyFunction() {
            @Override
            public int max(int a, int b) {
                return a > b ? a : b;
            }
        };
        int value = f.max(3, 5);
    }
}
```

위 코드의 메서드 `max()` 메서드를 람다식으로 아래와 같이 표현할 수 있다.

```java
class Example {
    public static void main(String[] args) {
        MyFunction f = (int a, int b) -> a > b ? a : b;
        int value = f.max(3, 5);
    }
}
```

이처럼 `MyFunction` 인터페이스를 구현한 익명 클래스의 객체를 람다식으로 대체할 수 있는 이유는 구현한 인터페이스가 함수형 인터페이스이기 때문이다.  
함수형 인터페이스가 되기 위한 조건은 default 메서드나 static 메서드를 가질 수 있지만, 구현해야 할 추상 메서드는 하나만 존재해야 한다.  
결국 구현해야 할 추상 메서드가 하니이기 때문에 람다식을 통해 익명 클래스의 객체를 생성할 수 있는 것이다.

### @FunctionalInterface

`@FunctionalInterface` 어노테이션은 추상 메서드가 하나만 존재하는지 컴파일러가 체크하도록 해주는 역할을 하여 함수형 인터페이스를 올바르게 정의했는지 확인할 수 있다.  
아래는 실제 Comparator 인터페이스의 정의이다.

```java

@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    
    // ...

    default Comparator<T> reversed() {
        // ... 구현 내용
    }

    // 그 외 default / static 메서드
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

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)