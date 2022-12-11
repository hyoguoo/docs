# String 클래스

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

### Constant Pool

> JVM이 시작될 때 생성되는 메모리 공간

자바 소스파일에 포함된 모든 문자열 리터럴은 컴파일 시에 클래스 파일에 저장되는데,  
이 때 같은 내용의 문자열 리터럴은 하나의 문자열 인스턴스를 공유하게 된다. 한 번 생성하면 변경할 수 없는 특성을 가지고 있기 때문이다.

String에 문자열을 할당하게 됐을 때 JVM은 아래와 같은 과정을 거친다.

1. `Constant Pool`에 같은 값이 있는지 탐색
2. 있으면 해당 인스턴스를 반환
3. 없으면 새로운 인스턴스를 생성하고 `Constant Pool`에 저장

만약 `new String()`을 통해 문자열을 생성하게 되면, `Constant Pool`에 저장되지 않고, 항상 새 객체를 생성하고 heap 영역에 저장하게 된다.  

###### 출처

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)