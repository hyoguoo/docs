---
layout: editorial
---

# Print(출력)

> 프로그램의 결과를 화면이나 파일로 보내는 것

## println

`println`은 변수의 리터럴 타입 그대로 출력하며, 출력 후 줄바꿈을 해주는 메서드이다.

```java
class Example {

    public static void main(String[] args) {

        System.out.println("Hello, World!"); // Hello, World! 출력 후 줄바꿈
        System.out.println(100); // 100 출력 후 줄바꿈
    }
}
```

## printf

`printf`는 지시자(specifier)를 통해 변수의 값을 여러 가지 형식으로 변환하여 출력하는 기능을 가지고 있다.

```java
class Example {

    public static void main(String[] args) {
        int value = 100;
        System.out.printf("Value: %d\n", value); // Value: 100 출력 후 줄바꿈
    }
}
```

### 자주 사용되는 지시자

| specifier |   description   |
|:---------:|:---------------:|
|    %b     |     boolean     |
|    %d     | decimal integer |
|    %o     |  octal integer  |
|  %x, %X   |  hexa-decimal   |
|    %f     | floating-point  |
|  %e, %E   |    exponent     |
|    %c     |    character    |
|    %s     |     string      |

이 지시자들을 사용하여 다양한 형식으로 데이터를 출력할 수 있다.

```java
class Example {

    public static void main(String[] args) {
        boolean flag = true;
        int dec = 255;
        double pi = 3.14159;

        System.out.printf("Boolean: %b\n", flag); // Boolean: true
        System.out.printf("Decimal: %d\n", dec); // Decimal: 255
        System.out.printf("Octal: %o\n", dec); // Octal: 377
        System.out.printf("Hexadecimal: %x\n", dec); // Hexadecimal: ff
        System.out.printf("Floating-point: %f\n", pi); // Floating-point: 3.141590
        System.out.printf("Character: %c\n", 'A'); // Character: A
        System.out.printf("String: %s\n", "Hello"); // String: Hello
    }
}
```

### 공간 지시자

공간 지시자는 출력될 값의 폭을 조정할 때 사용된다.

| specifier | result  |
|:---------:|:-------:|
|    %d     |  [10]   |
|    %5d    | [   10] |
|   %-5d    | [10   ] |
|   %05d    | [00010] |

```java
class Example {

    public static void main(String[] args) {
        int number = 10;
        System.out.printf("[%d]\n", number); // [10]
        System.out.printf("[%5d]\n", number); // [   10]
        System.out.printf("[%-5d]\n", number); // [10   ]
        System.out.printf("[%05d]\n", number); // [00010]
    }
}
```

### 진수 지시자

진수 지시자는 숫자를 다른 진법으로 출력할 때 사용된다.

| specifier | result  |
|:---------:|:-------:|
|    %x     |  fffff  |
|    %#x    | 0xfffff |
|    %#X    | 0XFFFFF |

```java
class Example {

    public static void main(String[] args) {
        int num = 1048575;
        System.out.printf("%x\n", num); // fffff
        System.out.printf("%#x\n", num); // 0xfffff
        System.out.printf("%#X\n", num); // 0XFFFFF
    }
}
```

- 10진수를 2진수로 출력해주는 지시자는 없기 때문에 `Integer.toBinaryString(intgerNumber)`를 사용할 수 있다.

```java

class Example {

    public static void main(String[] args) {
        int binaryNumber = 10;
        System.out.println(Integer.toBinaryString(binaryNumber)); // 1010
    }
}
```

- C언어에서처럼 char 타입을 정수로 출력할 수 없고, int 타입으로 형 변환하여 출력할 수 있다.

```java
class Example {

    public static void main(String[] args) {
        char ch = 'A';
        System.out.printf("%d\n", (int) ch); // 65
        System.out.printf("%d\n", ch); // java.util.IllegalFormatConversionException
    }
}
```

### 소수점 지시자

소수점 지시자는 실수 값을 출력할 때 소수점 이하 자릿수를 지정하는 데 사용된다.

`%{전체자리}.{소수점아래자리}f`

| 자릿수 | 1 | 2 | 3 | 4 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 |
|:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 결과  |   |   | 1 | . | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 | 0 |

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)
