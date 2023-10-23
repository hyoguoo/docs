---
layout: editorial
---

# String class

> 문자열을 다루는 클래스로, 문자열을 저장하는 데 사용되는 문자 배열을 내부적으로 가지고 있으며, 변경 불가능한 특성을 가진다.

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    @Stable
    private final byte[] value;
    private final byte coder;
    private int hash; // Default to 0
    // ...
}
```

때문에 문자열을 변경하는 경우, 인스턴스 변수의 값이 변경되는 것이 아니라, 새로운 문자열을 생성하게 되는 것이다.

## 문자열 비교

```java
class Example {

    public static void main(String[] args) {
        String str1 = "ogu"; // 리터럴로 생성
        String str2 = "ogu"; // 리터럴로 생성
        String str3 = new String("ogu"); // 인스턴스 생성
        String str4 = new String("ogu"); // 인스턴스 생성

        System.out.println(str1 == str2); // true
        System.out.println(str3 == str4); // false
        System.out.println(str1.equals(str2)); // true
        System.out.println(str1.equals(str3)); // true
    }
}
```

### String 클래스의 equals

- `String` 클래스의 `equals()` 메서드는 `Object` 클래스의 `equals()` 메서드를 오버라이딩한 것이다.
- `String` 클래스의 `equals()` 메서드는 두 객체의 내용이 같은지 비교한다.

```java
public class StringEqualsTest {
    public static void main(String[] args) {
        String s1 = "ogu";
        String s2 = "ogu";
        String s3 = new String("ogu");
        System.out.println(s1 == s2); // true
        System.out.println(s2 == s3); // false
        System.out.println(s2.equals(s3)); // true
    }


    // 실제 String 클래스의 equals() 메서드
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String) anObject;
            if (coder() == aString.coder()) {
                return isLatin1() ? StringLatin1.equals(value, aString.value)
                        : StringUTF16.equals(value, aString.value);
            }
        }
        return false;
    }
}
```

### Constant Pool

> JVM이 시작될 때 생성되는 메모리 공간

자바 소스파일에 포함된 모든 문자열 리터럴은 컴파일 시에 클래스 파일에 저장되는데,  
이 때 같은 내용의 문자열 리터럴은 하나의 문자열 인스턴스를 공유하게 된다. 한 번 생성하면 변경할 수 없는 특성을 가지고 있기 때문이다.

String에 문자열을 할당하게 됐을 때 JVM은 아래와 같은 과정을 거친다.

1. `Constant Pool`에 같은 값이 있는지 탐색
2. 있으면 해당 인스턴스를 반환
3. 없으면 새로운 인스턴스를 생성하고 `Constant Pool`에 저장

만약 `new String()`을 통해 문자열을 생성하게 되면, `Constant Pool`에 저장되지 않고, 항상 새 객체를 생성하고 heap 영역에 저장하게 된다.

## StringBuffer & StringBuilder

> 문자열을 변경할 수 있는 클래스

기존 String 클래스는 문자열을 변경할 때마다 새로운 인스턴스를 생성하고, 기존 인스턴스는 GC의 대상이 되기 때문에 문자열을 변경할 때는 `StringBuffer`나 `StringBuilder`를 사용하는 것이
좋다.

### StringBuffer

- 내부적으로 문자열 편집을 위한 `buffer`를 가지고 있음
- 생성자를 통해 초기 용량을 지정할 수 있으며 버퍼의 크기가 부족할 경우 자동으로 증가

#### StringBuffer 문자열 비교

StringBuffer 클래스의 equals 메서드는 오버라이딩하지 않아 Object 클래스의 equals 메서드(==)를 사용하게 된다.  
담고있는 문자열을 반환하는 `toString()`은 오버라이딩 되어 있어 해당 메서드를 통해 비교가 가능하다.

```java
public class StringBufferTest {
    public static void main(String[] args) {
        StringBuffer sb1 = new StringBuffer("ogu");
        StringBuffer sb2 = new StringBuffer("ogu");
        System.out.println(sb1 == sb2); // false
        System.out.println(sb1.equals(sb2)); // false
        System.out.println(sb1.toString().equals(sb2.toString())); // true
    }
}
```

### StringBuilder

- `thread safe`한 StringBuffer와 달리 `thread unsafe`
- 위 기능을 제외하고는 StringBuffer와 동일

### 비교

- String : 불변하기 때문에 문자열 연산이 적은 경우 사용
- StringBuffer : 문자열 연산이 많고 멀티쓰레드인 경우 사용
- StringBuilder : 문자열 연산이 많고 단일쓰레드이거나 동기화를 고려하지 않아도 될 경우 사용

|      -       |   String    | StringBuffer | StringBuilder |
|:------------:|:-----------:|:------------:|:-------------:|
|   storage    | String Pool |     Heap     |     Heap      |
|    modify    |      X      |      O       |       O       |
| thread safe  |      O      |      O       |       X       |
| synchronized |      O      |      O       |       X       |

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)