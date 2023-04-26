# 출력

## println

변수의 리터럴 타입 그대로 출력하며 출력 후 줄바꿈을 해준다.

## printf

지시자(specifier)를 통해 변수의 값을 여러 가지 형식으로 변환하여 출력하는 기능을 가지고 있다.

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

### 공간 지시자
| specifier | result  |
|:---------:|:-------:|
|    %d     |  [10]   |
|    %5d    | [   10] |
|   %-5d    | [10   ] |
|   %05d    | [00010] |

### 진수 지시자

| specifier | result  |
|:---------:|:-------:|
|    %x     |  fffff  |
|    %#x    | 0xfffff |
|    %#X    | 0XFFFFF |

10진수를 2진수로 출력해주는 지시자는 없기 때문에 `Integer.toBinariyString(intgerNumber)`를 사용하면 된다. 또한 C언어에서 처럼 char 타입을 정수로 출력할 수 없고, int
타입으로 형 변환하여 출력할 수 있다.

### 소수점 지시자

`14.10f`(`%{전체자리}.{소수점아래자리}f`)

| 자릿수 |  1  |  2  |  3  |  4  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |  9  |  0  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 결과  |     |     |  1  |  .  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |  9  |  0  |  0  |

###### 참고자료

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)