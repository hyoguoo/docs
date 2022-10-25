# 변수

```
단 하나의 값을 저장할 수 있는 메모리 공간
```

## 변수 명명 규칙

1. 대소문자 구분
2. 길이 제한 없음
3. 예약어 사용 불가능
4. 숫자로 시작 불가
5. `_` `$`를 제외한 특수문자 불가능

** 예약어란 프로그래밍 언어의 `int`, `finally`와 같은 구문에 사용되는 단어를 뜻한다.

## 변수의 타입

- 기본형(primitive type): 실제 값 저장
    - 논리형: boolean...
    - 문자형: char...
    - 숫자
        - 정수형: byte, shore, int, long...
        - 실수형: float, double...
- 참조형(reference type): 값이 저장되어 있는 주소값 저장

## 변수와 상수

- 변수(variable): 하나의 값을 저장하기 위한 공간
- 상수(constant): 값을 한 번만 저장할 수 있는 공간
- 리터럴(literal): 그 자체로 값을 의미하는 것

```java
int age=14; // 변수
final int OGU_NUMBER=59; // 상수
```

### String Pool

`String`은 자바에서 가장 자주 사용되는 객체이다. 그래서 일반 객체와는 다르게 특별한 Pool이 존재한다.  
Java에서는 문자열의 불변성 법칙 때문에 JVM에서 각 리터럴 문자열을 하나만 저장하여 문자열에 할당 된 메모리 양을 최적화 할 수 있다.  
String에 문자열을 할당하게 되면, JVM은 `String Pool`에 같은 값이 있는지 탐색하고,  
같은 값을 찾았을 경우 Java Compiler는 추가적으로 할당을 하지 않으면서 해당 값을 가진 메모리 주소를 반환한다.  
만약 찾지 못 했을 경우에는 새 값을 할당 한 뒤 그 주소를 반환하게 된다.

만약 `new String()`을 통해 생성자를 통한 할당을 하게 되면 Java Compiler는 새 객체를 생성하고 heap 공간에 할당하게 된다.  
그렇게 되면 모든 생성 마다 각 각 다른 장소의 메모리 주소를 가지게 될 것이다.  
그래서 아래와 같이 생성자를 통해 생성했을 경우에 같은 문자열을 가진 값이더라도 다른 주소값을 가지게 되어 논리 연산 결과가 `false` 가 반환하게 된다.

```java
String s1="ogu";
String s2="ogu";
String s3=new String("ogu");
System.out.println(s1==s2); // true
System.out.println(s2==s3); // false
```

## 리터럴 타입

변수와 마찬가지로 값 자체를 의미하는 리터럴에도 타입이 존재한다.  
논리형(treu/false), 문자형, 문자열에는 존재하지 않으며 정수형과 실수형에 존재한다.

- 정수타입 리터럴

|     Type      |  Expression  |
|:-------------:|:------------:|
|    binary     |  0b{number}  |
|     octal     |  0{number}   |
|  hexadecimal  |  0x{number}  |
|     `int`     |   {number}   |
|    `long`     | {number}l(L) |

** 소문자 `l`의 경우 숫자 `1`과 혼동을 일으키므로 `L` 사용을 추천

- 실수타입 리터럴

|     Type     |  Expression  |
|:------------:|:------------:|
|   `float`    | {number}f(F) |
|   `double`   | {number}d(D) |

**double 타입은 생략 가능

- 문자타입 리터럴

|   Type   |      Expression      |
|:--------:|:--------------------:|
|  `char`  | '{single character}' |
| `string` |      "{string}"      |

- 논리타입 리터럴

|   Type    | Expression |
|:---------:|:----------:|
| `boolean` | true/false |

### 리터럴 타입 할당

```java
float pi=3.14;          // 변수와 리터럴 타입 불 일치 -> error
double rate=59.59;       // double 리터럴 타입 생략 가능 -> OK
int i='A';            // 문자 'A'의 아스키코드인 65가 할당 -> OK
long d=59;             // 저장 범위가 넓은 타입에 좁은 타입 저장 가능 -> OK
int ii=0x123456789;   // int 타입의 범위를 넘는 값 저장 -> error
String str="";           // 빈 문자열 할당 -> OK
char ch='';            // 빈 문자 할당, 하나의 문자 반드시 필요 -> error
char chch=' ';         // 공백 문자 할당 가능 -> OK
String num1=7+7+"";  // 7 + 7 + "" -> 14 + "" -> "14"
String num2=""+7+7;  // "" + 7 + 7 -> "7" + 7 -> "77"
// `문자열 + any type -> 문자열 + 문자열 -> 문자열`
```

###### 출처

- Java의 정석
- https://www.javatpoint.com/string-pool-in-java
