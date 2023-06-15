# 배열

> 어떤 메모리 블록에 연속적으로 나열된 같은 유형의 변수 모음

연결 리스트와 비슷하게 선형적으로 저장하지만 원하는 인덱스를 알고 있을 경우 시간 복잡도의 차이가 있다.  
배열은 동적인 자료구조가 아니며 유한하고 고정된 수의 원소로 이루어지기 때문에 일부만 사용하더라도 배열에 있는 모든 원소에 대해 메모리가 할당되어야 한다.

- 시간 복잡도

|    |  배열  | 연결 리스트 |
|:--:|:----:|:------:|
| 탐색 | O(1) |  O(n)  |
| 삽입 | O(n) |  O(1)  |

## 배열 선언과 생성

```java
class Example {
    public void example() {
        int[] arr1; // 배열 선언
        arr1 = new int[5]; // 배열 생성
        int[] arr2 = new int[5]; // 선언과 생성 동시
        int[] arr3 = {1, 2, 3, 4, 5}; // 선언과 생성, 초기화 동시
    }
}
```

- 배열 선언: 생성된 배열을 다루기 위한 참조변수 공간이 생성
- 배열 생성: 배열의 크기를 지정되면서 배열의 각 원소에 대한 메모리 공간이 생성
- 초기화: 배열의 각 원소에 값을 할당

생성만 한 뒤 값을 할당하지 않은 배열은 각 원소의 기본값으로 초기화된다.

|          type          | default value |
|:----------------------:|:-------------:|
|        boolean         |     false     |
|          char          |      '0'      |
| byte, short, int, long |       0       |
|     float, double      |      0.0      |
|         object         |     null      |

## 배열 참조와 복사

단순한 대입만으로 배열 원소를 복사할 수 없으며 두 배열의 유형이 같은 경우 한 레퍼런스를 다른 레퍼런스에 대입할 수 있다.

```java
class Example {
    public void example() {
        int[] arrayA = new int[10];
        int[] arrayB = new int[10];
        arrayA = arrayB; // arrayA와 arrayB가 같은 배열 참조
    }
}
```

만약 한 배열의 내용을 다른 배열로 복사하고 싶은 경우 `System.arraycopy` 메소드를 사용하거나 반복문을 통해 복사할 수 있다.

```java
class Example {
    public void example() {
        int[] arrayA = new int[10];
        int[] arrayB = new int[10];
        System.arraycopy(arrayA, 0, arrayB, 0, arrayA.length);
    }
}
```

## [문자열](string_class.md)

> 문자들이 연속적으로 나열되어 있는 것  
> char 배열에 여러 가지 기능을 추가한 `String` 클래스를 이용해서 처리하게 된다.

문자열에 있는 개별 문자는 직접 엑세스 할 수 없고 특정 메서드를 써야 엑세스할 수 있으며, 문자열은 변형 불가능(`immutable`)하고, 조작하는 메서드는 새로운 문자열 인스턴스를 반환하는 형식이다.

## 주요 메서드

|                   메서드                    |                  설명                   |
|:----------------------------------------:|:-------------------------------------:|
|           `charAt(int index)`            |        문자열에서 index 위치의 문자를 반환         |
|              `int length()`              |              문자열의 길이를 반환              |
|   `String substring(int from, int to)`   |        문자열에서 해당 범위에 있는 문자열 반환         |
|       `String substring(int from)`       |       문자열에서 해당 위치부터 끝까지 문자열 반환        |
|       `String concat(String str)`        |          문자열을 더해서 새로운 문자열 반환          |
| `String replace(String old, String new)` | 문자열에서 old 문자열을 new 문자열로 바꾼 새로운 문자열 반환 |
|          `String toLowerCase()`          |        문자열을 소문자로 바꾼 새로운 문자열 반환        |
|          `String toUpperCase()`          |        문자열을 대문자로 바꾼 새로운 문자열 반환        |
|             `String trim()`              |      문자열의 앞뒤 공백을 제거한 새로운 문자열 반환       |
|      `String[] split(String regex)`      |     문자열을 regex를 기준으로 나눈 문자열 배열 반환     |
|       `boolean equals(Object obj)`       |              문자열이 같은지 비교              |
|       `int compareTo(String str)`        |             문자열을 사전순으로 비교             |
|        `int indexOf(String str)`         |       문자열에서 str이 처음 나타나는 위치를 반환       |
|   `boolean startsWith(String prefix)`    |         문자열이 prefix로 시작하는지 확인         |
|    `boolean endsWith(String suffix)`     |         문자열이 suffix로 끝나는지 확인          |
|      `boolean contains(String str)`      |          문자열이 str을 포함하는지 확인           |
|           `boolean isEmpty()`            |             문자열이 비어있는지 확인             |
|          `char[] toCharArray()`          |            문자열을 문자 배열로 변환             |

## 다차원 배열

```java
class Example {
    public void example() {
        int[][] arr1 = new int[3][4]; // 선언 및 생성
        int[][] arr2 = {{1, 2, 3}, {4, 5, 6}}; // 초기화
        int[][] arr3 = new int[3][]; // 가변 배열 선언 및 생성
        arr3[0] = new int[3];
        arr3[1] = new int[2];
        arr3[2] = new int[4];
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)