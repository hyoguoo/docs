# 스트림(Stream)

> 데이터 소스를 추상화하고, 다루는데 자주 사용되는 메서드를 정의한 인터페이스

데이터 소스가 무엇이던 간에 같은 방식으로 다룰 수 있도록 해주며 코드의 재사용성을 높여준다.

```java
class Example {
    public static void main(String[] args) {
        String[] array = {"a", "b", "c", "d", "e"};
        List<String> list = Arrays.asList(array);

        // 정렬된 대상 출력
        Arrays.sort(array);
        for (String string : array) {
            System.out.println(string);
        }

        Collections.sort(list);
        for (String string : list) {
            System.out.println(string);
        }

        // 스트림 생성
        Stream<String> arrayStream = Arrays.stream(array);
        String<String> listStream = list.stream();

        // 정렬된 대상 출력
        arrayStream.sorted().forEach(System.out::println);
        listStream.sorted().forEach(System.out::println);
    }
}
```

두 `Stream` 데이터 소스는 다르지만, 같은 방식으로 정렬하고 출력할 수 있다.  
또한 데이터 소스를 변경하지 않고도 정렬된 결과를 얻을 수 있다.

## 스트림의 특징

### 데이터 소스 불변

데이터 소스를 변경하지 않고 새로운 데이터 소스를 생성한다.

```java
class Example {
    public static void main(String[] args) {
        List<String> sortedList = list.stream().sorted().collect(Collectors.toList());
    }
}
```

### 내부 반복

간결한 코드 작성이 가능해진 이유로, 반복문을 메서드의 내부에 숨길 수 있다.

```java
class Example {
    public static void main(String[] args) {
        // 외부 반복
        for (String string : list) {
            System.out.println(string);
        }

        // 내부 반복
        list.stream().forEach(System.out::println);
    }
}
```

### 중간 연산과 최종 연산

- 중간 연산: 스트림을 반환하는 연산으로, 스트림에 연속해서 여러 개의 중간 연산을 호출할 수 있다.
- 최종 연산: 스트림을 반환하지 않는 연산으로, 스트림에 최종 연산을 호출하면 스트림을 소모하게 된다.

```java
class Example {
    public static void main(String[] args) {
        Stream<String> stream = list.stream();
        stream.sorted(); // 중간 연산
        stream.forEach(System.out::println); // 최종 연산
    }
}
```

### 일회성

일회성으로, 한 번 사용하면 더 이상 사용할 수 없다. 최종 연산을 호출하면 스트림을 소모하게 되며, 이후에는 스트림을 다시 사용할 수 없다.  
여기서 데이터 소스는 변경되거나 사라지지 않는다.

```java
class Example {
    public static void main(String[] args) {
        Stream<String> stream = list.stream();
        stream.sorted().forEach(System.out::println);
        stream.sorted().forEach(System.out::println); // IllegalStateException
    }
}
```

### 지연 연산

스트림의 최종 연산이 호출되기 전까지는 중간 연산이 실제로 수행되지 않는다.

## 스트림 생성

### 컬렉션

컬렉션의 최고 조상인 `Collection` 인터페이스에는 `stream()` 메서드가 정의되어 있다. 때문에 `Collection`의 자손 클래스인 `List`, `Set`, `Queue` 등은
모두 `stream()` 메서드를 사용할 수 있다.

### 배열

`Arrays` 클래스에는 배열을 스트림으로 변환하는 `stream()` 메서드가 정의되어 있다.  
또한 `int`, `long`, `double` 배열을 스트림으로 변환하는 `stream()` 메서드도 정의되어 있다.

### 특정 범위의 정수

`IntStream`와 `LongStream` 클래스에는 특정 범위의 정수를 스트림으로 변환하는 `range()` 메서드가 정의되어 있다.

```java
class Example {
    public static void main(String[] args) {
        IntStream.range(1, 10).forEach(System.out::println);
    }
}
```

### 난수

`Random` 클래스에는 난수를 생성하는 `ints()`, `longs()`, `doubles()` 메서드가 정의되어 있다.

```java
import java.util.Random;

class Example {
    public static void main(String[] args) {
        new Random().ints(); // 무한 스트림
        new Random().ints(1, 10); // 1 이상 10 미만의 난수를 무한 스트림
    }
}
```

### 람다식

`Stream` 클래스에는 람다식을 사용하여 스트림을 생성하는 `iterate()`, `generate()` 메서드가 정의되어 있다.  
`generate`는 `iterate`와 달리, 이전 요소를 사용하지 않는 차이가 있다.

```java
class Example {
    public static void main(String[] args) {
        Stream.iterate(1, n -> n + 2).limit(10).forEach(System.out::println);
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
    }
}
```

`iterate()`와 `generate()`에 의해 생성된 스트림은 기본형 스트림 타입의 참조변수로 다룰 수 없고, 적용 시키기
위해선 `mapToInt()`, `mapToLong()`, `mapToDouble()` 메서드를 사용해야 한다.

```java
class Example {
    public static void main(String[] args) {
        IntStream intStream = Stream.iterate(1, n -> n + 2).limit(10).mapToInt(Integer::intValue);
        intStream.forEach(System.out::println);
    }
}
```

### 빈 스트림

`Stream` 클래스에는 빈 스트림을 생성하는 `empty()` 메서드가 정의되어 있다.

```java
class Example {
    public static void main(String[] args) {
        Stream.empty().forEach(System.out::println);
    }
}
```

### 스트림 연결

`Stream` 클래스에는 두 개의 스트림을 연결하는 `concat()` 메서드가 정의되어 있다.

```java

class Example {
    public static void main(String[] args) {
        Stream.concat(Stream.of(1, 2, 3), Stream.of(4, 5, 6)).forEach(System.out::println);
    }
}
```

## 스트림의 중간 연산

- `skip()` / `limit()`

스트림의 요소를 건너뛰거나, 제한하는 연산

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).skip(2).limit(2).forEach(System.out::println);
    }
}
```

- `filter()` / `distinct()`

스트림의 요소를 걸러내거나, 중복을 제거하는 연산

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5, 1, 2, 3).filter(i -> i % 2 == 0).distinct().forEach(System.out::println);
    }
}
```

- `sorted()`

스트림의 요소를 정렬하는 연산

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).sorted().forEach(System.out::println);
    }
}
```

- `map()`

스트림의 요소를 다른 요소로 대체하는 연산

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).map(i -> i * 2).forEach(System.out::println);
    }
}
```

- `peek()`

`forEach()`와 비슷하지만, 스트림의 요소를 소모하지 않는 연산

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).peek(System.out::println).forEach(System.out::println);
    }
}
```

- `mapToInt()`, `mapToLong()`, `mapToDouble()`

스트림의 요소를 기본형 스트림으로 변환하는 연산

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).mapToInt(Integer::intValue).forEach(System.out::println);
    }
}
```

- `flatMap()`

스트림의 요소를 다른 스트림으로 변환하는 연산

```java
class Example {
    public static void main(String[] args) {
        String[] lineArr = {
                "Believe or not It is true",
                "Do or do not There is no try"
        };

        Stream<String> lineStream = Stream.of(lineArr);
        Stream<Stream<String>> strArrStream = lineStream.map(line -> Stream.of(line.split(" +")));
        strArrStream.map(strStream -> Arrays.toString(strStream.toArray(String[]::new))).forEach(System.out::println);
        // [[Believe, or, not, It, is, true], [Do, or, do, not, There, is, no, try]]

        lineStream = Stream.of(lineArr);
        Stream<String> stream = lineStream.flatMap(line -> Stream.of(line.split(" +")));
        stream.forEach(System.out::println);
        // [Believe, or, not, It, is, true, Do, or, do, not, There, is, no, try]
    }
}
```

## 스트림의 최종 연산

스트림의 요소를 소모해 결과를 만들어내고, 최종 연산 후에는 스틀미이 닫히게 되어 더 이상 사용할 수 없다.  
결과로는 단일 값이거나 스트림의 요소가 담긴 배열 또는 컬렉션일 수 있다.

- `forEach()`

스트림의 요소를 소모하면서 지정된 람다식을 수행하는 연산  
`void forEach(Consumer<? super T> action)`

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).forEach(System.out::println);
    }
}
```

- 조건 검사 연산(`allMatch()`, `anyMatch()`, `noneMatch()`)

스트림의 요소에 대해 지정된 조건에 맞는지 검사하는 연산으로 `boolean` 값을 반환한다.

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).allMatch(i -> i > 0); // true
        Stream.of(1, 2, 3, 4, 5).anyMatch(i -> i > 0); // true
        Stream.of(1, 2, 3, 4, 5).noneMatch(i -> i > 0); // false
    }
}
```

- `findFirst()`, `findAny()`

스트림의 요소 중에서 첫 번째 요소를 반환하는 연산으로 `Optional` 객체를 반환한다.

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).findFirst(); // Optional[1]
        Stream.of(1, 2, 3, 4, 5).findAny(); // Optional[1]
    }
}
```

- `count()`, `max()`, `min()`, `sum()`, `average()`

스트림의 요소의 개수, 최대값, 최소값, 합계, 평균을 구하는 연산으로 `Optional` 객체를 반환한다.

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).count(); // 5
        Stream.of(1, 2, 3, 4, 5).max(Integer::compareTo); // Optional[5]
        Stream.of(1, 2, 3, 4, 5).min(Integer::compareTo); // Optional[1]
        Stream.of(1, 2, 3, 4, 5).sum(); // 15
        Stream.of(1, 2, 3, 4, 5).average(); // Optional[3.0]
    }
}
```

- `reduce()`

스트림의 요소를 하나씩 소모하면서 지정된 람다식을 수행하는 연산으로 `Optional` 객체를 반환한다.

```java
class Example {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5).reduce((a, b) -> a + b); // Optional[15]
    }
}
```

위와 같이 `reduce()`를 사용하면 `sum()`과 같은 결과를 얻을 수 있으며 내부적으로도 `reduce()`를 통해 구현되어 있다.

- `collect()`

    - `collect()`: 스트림의 최종 연산, 매개변수로 `Collector`를 받음
    - `Collector`: collect()의 매개변수로 사용되는 인터페이스, 스트림의 요소를 소모하면서 어떤 작업을 수행할지 정의
    - `Collectors`: Collector를 구현한 클래스
        - `toList()`, `toSet()`, `toCollection()`, `toMap()` 등이
          존재([문서](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) 참고)
        - 그 외에도 직접 구현하여 사용할 수 있다.

###### 참고자료

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)