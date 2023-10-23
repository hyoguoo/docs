---
layout: editorial
---

# Iterator

Iterator 인터페이스를 사용하여 Java 컬렉션 데이터를 순회하고 접근할 수 있다.  
Iterator는 컬렉션 내의 요소를 반복적으로 순회하면서 데이터를 읽을 수 있는 메커니즘을 제공하고, 컬렉션의 내부 구조를 알 필요 없이 데이터에 접근할 수 있도록 해준다.

## Iterator 인터페이스의 주요 메서드

Iterator 인터페이스는 다음과 같은 메서드를 가지고 있다.

- `boolean hasNext()`: 다음 요소가 존재하는지 확인하는 메서드로, 다음 요소가 있으면 `true`를 반환하고, 더 이상 순회할 요소가 없으면 `false`를 반환
- `E next()`: 다음 요소를 반환하는 메서드, (호출할 다음 요소가 없을 경우 `NoSuchElementException` 예외가 발생)

```java
public class IteratorExample {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");

        // Iterator 생성
        Iterator<String> iterator = names.iterator();

        // 순회하면서 출력
        while (iterator.hasNext()) {
            String name = iterator.next();
            System.out.println(name);
        }
    }
}
```

# 스트림

> 데이터 소스를 추상화하고, 다루는데 자주 사용되는 메서드를 정의한 인터페이스

Java 8 이후에는 스트림(Stream) API가 도입되어 컬렉션 데이터를 처리하는 더 간결하고 표현력 있는 방법을 제공한다.  
스트림은 컬렉션 요소를 람다식을 활용하여 처리할 수 있으며, Iterator보다 많은 연산을 지원한다.  
마찬가지로 스트림을 사용하면 데이터 소스가 무엇이든 같은 방식으로 다룰 수 있도록 해주며 코드의 재사용성을 높여준다.

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

두 개의 Stream 데이터 소스는 다르지만, 같은 방식으로 정렬하고 출력할 수 있고, 또한 데이터 소스를 변경하지 않고도 정렬된 결과를 얻을 수 있게 된다.

## 스트림의 특징

### 데이터 소스 불변

스트림은 데이터 소스를 변경하지 않고 새로운 데이터 소스를 생성한다.

```java
class Example {
    public static void main(String[] args) {
        List<String> sortedList = list.stream().sorted().collect(Collectors.toList());
    }
}
```

### 내부 반복

스트림을 사용하면 반복문을 메서드의 내부에 숨길 수 있어 코드가 간결해 진다.

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
여기서 데이터 소스는 불변성 특징을 가지기 때문에, 소스는 변경되거나 사라지지 않는다.

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

- `Collection` 인터페이스: `stream()` 메서드가 정의되어 있어 컬렉션을 스트림으로 변환할 수 있다.(자손 클래스 모두 사용 가능)
- `Arrays` 클래스: 배열을 스트림으로 변환하는 `stream()` 메서드가 정의되어 있어 배열을 스트림으로 변환할 수 있다.
- `Map` 인터페이스: `keySet()`, `values()`, `entrySet()` 메서드를 사용하여 스트림으로 변환할 수 있다.

결국 모든 데이터 소스는 `stream()` 메서드를 사용하여 스트림으로 변환할 수 있게 된다.

```java
class Example {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("a", "b", "c", "d", "e");
        Stream<String> stream1 = list.stream();

        String[] array = {"a", "b", "c", "d", "e"};
        Stream<String> stream2 = Arrays.stream(array);

        Map<String, Integer> map = new HashMap<>() {{
            put("a", 1);
            put("b", 2);
            put("c", 3);
        }};
        Stream<Map.Entry<String, Integer>> streamEntry = map.entrySet().stream();
        Stream<String> streamKey = map.keySet().stream();
        Stream<Integer> streamValue = map.values().stream();
    }
}
```

그 외에 아래와 같이 여러 가지 방법으로 스트림을 생성할 수 있다

```java
class StreamCreationExample {
    public static void main(String[] args) {
        // 특정 범위의 정수 스트림 생성
        IntStream range = IntStream.range(1, 10);

        // 난수 스트림 생성
        IntStream randomStream = new Random().ints(1, 10).limit(10);

        // 람다식을 사용하여 스트림 생성
        Stream<Integer> iterateStream = Stream.iterate(1, n -> n + 2).limit(10); // 람다식을 사용하여 스트림 생성
        Stream<Double> generateStream = Stream.generate(Math::random).limit(10); // Supplier 인터페이스를 사용하여 스트림 생성

        // 빈 스트림 생성
        Stream<Object> emptyStream = Stream.empty();

        // 두 스트림 연결
        Stream<Integer> concatStream = Stream.concat(Stream.of(1, 2, 3), Stream.of(4, 5, 6));
    }
}
```

## 스트림의 중간 연산

스트림의 중간 연산으로는 아래와 같은 것들이 있다.

```java
class IntermediateOperationsExample {
    public static void main(String[] args) {
        // skip()와 limit(): 요소 건너뛰기와 제한
        Stream.of(1, 2, 3, 4, 5)
                .skip(2)
                .limit(2)
                .forEach(System.out::println);

        // filter()와 distinct(): 요소 걸러내기와 중복 제거
        Stream.of(1, 2, 3, 4, 5, 1, 2, 3)
                .filter(i -> i % 2 == 0)
                .distinct()
                .forEach(System.out::println);

        // sorted(): 요소 정렬
        Stream.of(5, 2, 1, 4, 3)
                .sorted()
                .forEach(System.out::println);

        // map(): 요소 변환
        Stream.of(1, 2, 3, 4, 5)
                .map(i -> i * 2)
                .forEach(System.out::println);

        // peek(): 요소 확인 (소비하지 않음)
        Stream.of(1, 2, 3, 4, 5)
                .peek(System.out::println)
                .forEach(System.out::println);

        // mapToInt(), mapToLong(), mapToDouble(): 기본형 스트림으로 변환
        Stream.of(1, 2, 3, 4, 5)
                .mapToInt(Integer::intValue)
                .forEach(System.out::println);

        // flatMap(): 스트림의 요소를 다른 스트림으로 변환하는 연산
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

```java
class TerminalOperationsExample {

    public static void main(String[] args) {
        // forEach(): 스트림 요소를 소모하면서 작업 수행
        Stream.of(1, 2, 3, 4, 5).forEach(System.out::println);

        // allMatch(), anyMatch(), noneMatch(): 요소에 대해 지정된 조건에 맞는지 검사하는 연산으로 boolean 값을 반환
        Stream.of(1, 2, 3, 4, 5).allMatch(i -> i > 0); // true
        Stream.of(1, 2, 3, 4, 5).anyMatch(i -> i > 2); // true
        Stream.of(1, 2, 3, 4, 5).noneMatch(i -> i > 0); // false

        // findFirst()와 findAny(): 스트림의 요소 중에서 찾은 첫 번째 요소를 `Optional` 객체로 반환(찾지 못하면 empty 상태의 `Optional` 객체 반환)
        Stream.of(1, 2, 3, 4, 5).findFirst(); // Optional[1]
        Stream.of(1, 2, 3, 4, 5).findAny(); // Optional[1]

        // reduce(): 스트림의 요소를 하나씩 소모하면서 지정된 람다식을 수행하는 연산으로 `Optional` 객체를 반환
        // reduce()를 사용하면 sum(), max(), min() 등의 집계 연산을 구현할 수 있다.
        Stream.of(1, 2, 3, 4, 5).reduce((a, b) -> a + b); // Optional[15]

        // count(), max(), min(): 스트림의 요소를 소모하면서 지정된 연산을 수행하고, 최종 결과를 반환
        Stream.of(1, 2, 3, 4, 5).count(); // 5
        Stream.of(1, 2, 3, 4, 5).max(Integer::compareTo); // Optional[5]
        Stream.of(1, 2, 3, 4, 5).min(Integer::compareTo); // Optional[1]

        // collect(): 스트림의 요소를 수집하는 연산으로, `Collector`를 매개변수로 받아 받은 타입으로 변환하여 반환
        List<Integer> list = Stream.of(1, 2, 3, 4, 5).collect(Collectors.toList());
        Set<Integer> set = Stream.of(1, 2, 3, 4, 5).collect(Collectors.toSet());
    }
}
```

### findFirst() vs findAny()

`findFirst()`와 `findAny()` 두 메서드 모두 찾은 첫 번째 요소를 `Optional` 객체로 반환한다.  
하지만 Parallel Stream을 사용하여 병렬로 처리를 할 때는 두 메서드에 아래와 같은 차이가 있다.

- `findFirst()`: 여러 요소가 조건에 부합해도 Stream의 순서를 고려하여 첫 번째 요소를 반환
- `findAny()`: Multiple Thread에서 여러 요소를 처리하다가 가장 먼저 찾은 요소를 반환

```java
class FindFirstAndFindAnyExample {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        Optional<Integer> findFirst = list.stream().parallel().filter(i -> i % 2 == 0).findFirst();
        Optional<Integer> findAny = list.stream().parallel().filter(i -> i % 2 == 0).findAny();

        System.out.println(findFirst.get()); // 2
        System.out.println(findAny.get()); // 2, 4, 6, 8, 10 중 하나
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)