---
layout: editorial
---

# Variable(변수)

> 단 하나의 값을 저장할 수 있는 메모리 공간

## 변수 명명 규칙

1. 대소문자 구분
2. 길이 제한 없음
3. 예약어 사용 불가능
4. 숫자로 시작 불가
5. `_` `$`를 제외한 특수문자 불가능

** 예약어: 프로그래밍 언어의 `int`, `finally`와 같은 구문에 사용되는 단어

## 변수의 타입

변수는 크게 원시 타입과 참조 타입으로 나뉜다.

### 원시 타입(primitive type)

실제 값을 저장하며, null 값을 가질 수 없다.

| 타입  | 설명                     | 크기               | 예시                   |
|-----|------------------------|------------------|----------------------|
| 논리형 | boolean                | 1 byte           | true, false          |
| 문자형 | char                   | 2 bytes          | 'A', 'b', '$'        |
| 정수형 | byte, short, int, long | 1, 2, 4, 8 bytes | 42, -1000, 987654321 |
| 실수형 | float, double          | 4, 8 bytes       | 3.14, -0.001         |

### 참조 타입(reference type)

값이 저장되어 있는 주소값 저장한다. 기본형을 객체로 다루어야 할 때가 있는데, 이 때 사용하는 [Wrapper 클래스](./wrapper-class)가 존재한다.

** 논리형이 1bit가 아닌 1byte인 이유:
[CPU can't address anything smaller than a byte.](https://stackoverflow.com/questions/4626815)

## 변수와 상수

변수와 상수는 값을 저장하는 점에서 동일하지만, 아래 차이점이 존재한다.

- 변수(variable): 하나의 값을 저장하기 위한 공간
- 상수(constant): 값을 한 번만 저장할 수 있는 공간
- 리터럴(literal): 그 자체로 값을 의미하는 것

```java
class Example {

    void example() {
        int age = 14; // 변수
        final int OGU_NUMBER = 59; // OGU_NUMBER: 상수, 59: 리터럴
    }
}
```

## 리터럴 타입

변수와 마찬가지로 값 자체를 의미하는 리터럴에도 타입이 존재한다.  
논리형(true/false), 문자형, 문자열에는 존재하지 않으며 정수형과 실수형에 존재한다.

### 정수타입 리터럴

|    Type     |  Expression  |
|:-----------:|:------------:|
|   binary    |  0b{number}  |
|    octal    |  0{number}   |
| hexadecimal |  0x{number}  |
|    `int`    |   {number}   |
|   `long`    | {number}l(L) |

** 소문자 `l`의 경우 숫자 `1`과 혼동을 일으킬 수 있기 때문에 대문자 `L`을 사용하는 것이 좋다.

### 실수타입 리터럴

|   Type   |  Expression  |
|:--------:|:------------:|
| `float`  | {number}f(F) |
| `double` | {number}d(D) |

** double 타입은 생략 가능

### 문자타입 리터럴

|   Type   |      Expression      |
|:--------:|:--------------------:|
|  `char`  | '{single character}' |
| `string` |      "{string}"      |

### 논리타입 리터럴

|   Type    | Expression |
|:---------:|:----------:|
| `boolean` | true/false |

### 리터럴 타입 할당 예시

```java
class Example {

    void example() {
        float pi = 3.14;          // 변수와 리터럴 타입 불 일치 -> error
        double rate = 59.59;       // double 리터럴 타입 생략 가능 -> OK
        int i = 'A';            // 문자 'A'의 아스키코드인 65가 할당 -> OK
        long d = 59;             // 저장 범위가 넓은 타입에 좁은 타입 저장 가능 -> OK
        int ii = 0x123456789;   // int 타입의 범위를 넘는 값 저장 -> error
        String str = "";           // 빈 문자열 할당 -> OK
        char ch = '';            // 빈 문자 할당, 하나의 문자 반드시 필요 -> error
        char chch = ' ';         // 공백 문자 할당 가능 -> OK
        String num1 = 7 + 7 + "";  // 7 + 7 + "" -> 14 + "" -> "14"
        String num2 = "" + 7 + 7;  // "" + 7 + 7 -> "7" + 7 -> "77"
        // `문자열 + any type -> 문자열 + 문자열 -> 문자열`
    }
}
```

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
- [java T point](https://www.javatpoint.com/string-pool-in-java)
