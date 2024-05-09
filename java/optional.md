---
layout: editorial
---

# Optional

자바에서는 원시 타입을 제외한 모든 것이 `null`이 될 수 있다.  
때문에 프로그래밍을 할 때 `NullPointerException`을 흔하게 마주치게 되는데, 제대로 처리하지 않으면 프로그램의 안정성이 떨어지고, 복잡한 코드가 생겨날 수 있다.

```java
class Example {
    public static void main(String[] args) {
        // case 1
        String str = "abc";
        System.out.println(str.toUpperCase()); // ABC
        str = null;
        System.out.println(str.toUpperCase()); // NullPointerException

        // case 2
        String str = "abc";
        if (str != null) {
            System.out.println(str.toUpperCase()); // ABC
        } else {
            System.out.println("null");
        }
    }
}
```

이러한 문제를 해결하기 위해 자바 8에서는 `Optional`을 도입하여 `null`을 더 안전하게 처리할 수 있도록 지원한다.

## Optional 객체 생성

`Optional`은 `null`이 될 수 있는 객체를 감싸는 래퍼 클래스로, `null`이 될 수 있는 객체를 담고 있는 `Optional` 객체를 생성할 수 있다.  
생성하는 방법으론 아래 세 가지가 있다.

```java
class Example {
    public static void main(String[] args) {
        Map<String, String> map = Map.of("existKey", "existValue");
        Optional<String> opt1 = Optional.of(map.get("existKey"));
        Optional<String> opt2 = Optional.ofNullable(map.get("notExistKey"));
        Optional<String> opt3 = Optional.empty();
    }
}
```

- `Optional.of(T value)`: `null`이 아닌 객체를 담고 있는 `Optional` 객체를 생성(만약 `null`이 넘어오면 `NullPointerException`을 발생)
- `Optional.ofNullable(T value)`: `null`이 될 수 있는 객체를 담고 있는 `Optional` 객체를 생성(만약 `null`이 넘어오면 `Optional.empty()` 반환)
- `Optional.empty()`: `null`을 담고 있는 `Optional` 객체 생성

## Optional 객체 조회

`Optional` 객체에 담긴 값을 가져오기 위해서는 기본적으로 `get()` 메서드를 사용해서 가져올 수 있다.

```java
class Example {
    public static void main(String[] args) {
        Optional<String> opt = Optional.of("abc");
        String str = opt.get();
        System.out.println(str); // abc
    }
}
```

`null` 체크를 하지 않고 바로 값을 가져오게 되면 해당 객체가 `null`일 수도 있기 때문에 체크를 해주는 것이 좋다.  
때문에 조회 후 `null` 체크를 하기 위해 아래와 같은 메서드들을 사용할 수 있다.

- `ifPresent()`: 값이 있는지 확인
- `ifPresent(Consumer<? super T> action)`: 값이 있으면 `Consumer`를 실행
- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`: 값이 있으면 `Consumer`를 실행, 없으면 `Runnable`을 실행
- `orElse(T other)`: 값이 없으면 전달 받은 인자를 반환
- `orElseGet(Supplier<? extends T> other)`: 값이 없으면 `Supplier` 인터페이스를 받아 실행한 결과를 반환
- `orElseThrow(Supplier<? extends X> exceptionSupplier)`: 값이 없으면 예외를 발생

```java
class Example {
    public static void main(String[] args) {
        Optional<String> opt = Optional.of("abc");

        // ifPresent: 값이 있는지 확인 -> 안티 패턴
        if (opt.isPresent()) {
            System.out.println(opt.get()); // abc
        }

        // ifPresent + 람다식
        Optional<String> opt1 = Optional.of("abc");
        opt1.ifPresent(s -> System.out.println(s.toUpperCase())); // ABC
        opt1 = Optional.empty();
        opt1.ifPresent(s -> System.out.println(s.toUpperCase())); // null

        // ifPresentOrElse + 람다식
        Optional<String> opt2 = Optional.of("abc");
        opt2.ifPresentOrElse(s -> System.out.println(s.toUpperCase()), () -> System.out.println("null")); // ABC
        opt2 = Optional.empty();
        opt2.ifPresentOrElse(s -> System.out.println(s.toUpperCase()), () -> System.out.println("null")); // null

        // orElse: 값이 없으면 기본값 설정
        Optional<String> opt3 = Optional.of("abc");
        System.out.println(opt3.map(String::toUpperCase).orElse("null")); // ABC
        opt3 = Optional.empty();
        System.out.println(opt3.map(String::toUpperCase).orElse("null")); // null

        // orElseGet: 값이 없을때 Supplier 인터페이스를 받아 실행하여 기본값 설정
        Optional<String> opt4 = Optional.of("abc");
        System.out.println(opt4.map(String::toUpperCase).orElseGet(() -> "null")); // ABC
        opt4 = Optional.empty();
        System.out.println(opt4.map(String::toUpperCase).orElseGet(() -> "null")); // null

        // orElseThrow: 값이 없을 때 예외 발생
        Optional<String> opt5 = Optional.of("abc");
        System.out.println(opt5.orElseThrow(IllegalArgumentException::new)); // abc
        opt5 = Optional.empty();
        System.out.println(opt5.orElseThrow(IllegalArgumentException::new)); // IllegalArgumentException
    }
}
```

###### 참고자료

- [실무 자바 개발을 위한 OOP와 핵심 디자인 패턴](https://school.programmers.co.kr/learn/courses/17778/17778-실무-자바-개발을-위한-oop와-핵심-디자인-패턴)
