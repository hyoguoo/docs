---
layout: editorial
---

# Number

숫자를 저장하는 타입은 값의 정확도에 따라 참값과 근삿값 타입으로 나눌 수 있다.

- 참값(Exact Value): 소수잠 이하 값의 유무와 관계없이 정확히 그 값을 그대로 저장하는 타입(INTEGER, *INT, DECIMAL)
- 근삿값(Approximate Value): 부동 소수점이 불리는 값을 의미하며, 처음 컬럼에 저장한 값과 조회된 값이 일치않고 최대한 비슷한 값을 저장하는 타입(FLOAT, DOUBLE)

또한 값이 저장되는 포맷에 따라 아래와 같이 나눌 수 있다.

- 이진 표기법(Binary): 프로그래밍 언어에서 사용하는 정수나 실수 타입을 의미하며, 적은 메모리나 디스크 공간에 저장할 수 있다.(INTEGER, BIGINT 등 대부분의 숫자 타입)
- 십진 표기법(DECIMAL): 숫자 값의 각 자릿값을 표현하기 위해 4비트나 한 바이트를 사용해서 저장하는 타입으로, 정확한 값을 저장할 수 있지만 메모리나 디스크 공간을 많이 사용한다.(DECIMAL만 해당)

## 정수

DECIMAL 타입을 제외하고 정수를 저장하는 타입은 5가지가 존재하며, 각 타입별 범위는 아래와 같다.

|  데이터 타입   | 저장공간(Bytes) |        최솟값(Signed)         | 최솟값(Unsigned) |        최댓값(Signed)        |       최댓값(Unsigned)        |
|:---------:|:-----------:|:--------------------------:|:-------------:|:-------------------------:|:--------------------------:|
|  TINYINT  |      1      |            -128            |       0       |            127            |            255             |
| SMALLINT  |      2      |          -32,768           |       0       |          32,767           |           65,535           |
| MEDIUMINT |      3      |         -8,388,608         |       0       |         8,388,607         |         16,777,215         |
|    INT    |      4      |       -2,147,483,648       |       0       |       2,147,483,647       |       4,294,967,295        |
|  BIGINT   |      8      | -9,223,372,036,854,775,808 |       0       | 9,223,372,036,854,775,807 | 18,446,744,073,709,551,615 |

UNSIGNED 옵션 사용은 인덱스의 사용 여부에 영향을 끼치진 않지만, 범위가 다르기 때문에 조인 시엔 각 타입별로 동일한 타입을 사용하는 것이 좋다.

## 부동 소수점

FLOAT, DOUBLE 타입에 해당하며, 숫자 값의 길이에 따라 유효 범위의 소수점 자릿수가 바뀌기 떄문에 정확한 유효 소수점 값을 식별하기 어렵다.  
떄문에 값을 따져서 비교하기가 쉽지 않고, 근삿값을 저장하는 방식이라서 동등 비교(Equal)는 사용할 수 없다.

## DECIMAL

부동 소수점에서 유효 범위 이외의 값은 가변적이므로 정확한 값을 보장할 수 없기 때문에 매우 중요한 값은 DECIMAL 타입으로 저장하는 것이 좋다.  
자릿수만큼 고정된 크기로 저장하기 때문에 메모리나 디스크 공간을 많이 사용하지만, 소수점에서 정확한 값을 보장할 수 있다.(정수를 사용하는 경우 INTEGER, BIGINT 타입을 사용하는 것이 좋다.)

###### 참고자료

- [Real MySQL 8.0 (2권)](https://kobic.net/book/bookInfo/view.do?isbn=9791158392727)
