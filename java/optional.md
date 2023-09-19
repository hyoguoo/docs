# Optional

자바에서는 원시 타입을 제외한 모든 것이 `null`이 될 수 있다.  
때문에 프로그래밍을 할 때 `NullPointerException`을 흔하게 마주치게 되는데, 이는 제대로 처리하지 못하면 프로그램의 안정성이 떨어지고, 복잡한 코드가 생겨날 수 있다.

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

```java
class Example {
    public static void main(String[] args) {
        Optional<String> opt = Optional.of("abc");
    }
}
```

`Optional`은 `null`이 될 수 있는 객체를 감싸는 래퍼 클래스로, `null`이 될 수 있는 객체를 담고 있는 `Optional` 객체를 생성할 수 있다.

## Optional을 사용한 값 가죠오기

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

```java
class Example {
    public static void main(String[] args) {
        // ifPresent: 값이 있는지 확인
        Optional<String> opt = Optional.of("abc");
        if (opt.isPresent()) {
            System.out.println(opt.get()); // abc
        }

        // ifPresent + 람다식
        Optional<String> opt2 = Optional.of("abc");
        opt2.ifPresent(s -> System.out.println(s.toUpperCase())); // ABC
        opt2 = Optional.empty();
        opt2.ifPresent(s -> System.out.println(s.toUpperCase())); // null


        // orElse: 값이 없을 때 기본값 설정
        Optional<String> opt1 = Optional.of("abc");
        System.out.println(opt1.map(String::toUpperCase).orElse("null")); // ABC
        opt1 = Optional.empty();
        System.out.println(opt1.map(String::toUpperCase).orElse("null")); // null


        // orElseThrow: 값이 없을 때 예외 발생
        Optional<String> opt3 = Optional.of("abc");
        System.out.println(opt3.orElseThrow(IllegalArgumentException::new)); // ABC
        opt3 = Optional.empty();
        System.out.println(opt3.orElseThrow(IllegalArgumentException::new)); // IllegalArgumentException
    }
}
```