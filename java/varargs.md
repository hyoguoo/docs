---
layout: editorial
---

# Varargs(가변인수)

JDK 5에서 도입된 기능으로 메서드의 매개변수 개수를 클라이언트가 조절할 수 있게 해준다.  
자바 라이브러리에서는 `String.format()`이 대표적인 예이다.

```java
class Main {
    public static void main(String[] args) {
        System.out.println(sum(1, 2, 3, 4, 5));
    }

    private static int sum(int... args) {
        int sum = 0;
        for (int arg : args) {
            sum += arg;
        }
        return sum;
    }
}
```

컴파일러가 컴파일 타임에 배열을 만들고, 전달 된 인자들을 배열로 래핑하여 전달하는 방식으로 동작한다.  
메서드 내부에서는 배열을 다루는 것과 완벽히 동일하게 동작한다.

## 주의사항

### 1. 마지막 인자로만 사용 가능

가변인수는 말 그대로 가변인수이기 때문에, 몇 개의 인자가 전달될지 모르는 상황이기 때문에, 마지막 인자로만 사용할 수 있다.

### 2. 오버로딩 주의

가변인수를 사용한 메서드와 동일한 시그니처를 가진 메서드를 오버로딩하면, 호출되는 메서드를 판단하기 모호해지고 컴파일 에러가 발생할 수 있다.

```java
class Main {
    public static void main(String[] args) {
        System.out.println(sum(1, 2, 3, 4));
    }

    // origin
    private static int sum(int... args) {
        int sum = 0;
        for (int arg : args) {
            sum += arg;
        }
        return sum;
    }

    // 인자가 3개인 경우 호출되지만, 모호함이 발생
    private static int sum(int a, int b, int c) {
        return a + b + c;
    }

    // 컴파일 에러 발생, Ambiguous method call. Both
    private static int sum(int a, int b, int... args) {
        int sum = 0;
        for (int arg : args) {
            sum += arg;
        }
        return sum + a + b;
    }
}
```

### 3. [Generic 사용 시](./effective_java/item32.md)
