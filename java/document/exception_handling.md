# 예외처리

> 예외처리는 프로그램이 실행되는 동안 발생하는 예외를 처리하는 것을 말한다.

- 프로그램 에러
    - 컴파일 에러: 컴파일 시 발생하는 에러
    - 런타임 에러: 실행 시 발생하는 에러
    - 논리적 에러: 실행은 되지만 의도와 다르게 동작하는 에러

- 예외 클래스 계층 구조
    - 모든 예외 클래스는 `java.lang.Throwable` 클래스를 상속받는다.
    - `Error` 클래스와 `Exception` 클래스는 `Throwable` 클래스를 상속받는다.
    - `Error` 클래스 : 프로그램 코드에 의해서 수습될 수 없는 심각한 오류
    - `Exception` 클래스 : 프로그램에서 코드에 의해서 수습될 수 있는 미약한 오류

![img.png](../image/throwable_class.png)

## try-catch-finally

- `try`: 예외가 발생할 수 있는 코드
- `catch`: 예외가 발생했을 때 수행할 코드
    - 프로그램의 비정상 종료를 막고, 정상적인 실행상태를 유지
- `finally`: 예외 발생 여부와 상관없이 항상 수행되어야 하는 코드
    - 보통 자원 해제 코드 작성

```java
class Example {
    public static void main(String[] args) {
        try {
            // 예외가 발생할 수 있는 코드
        } catch (Exception1 e1) {
            // Exception1 예외가 발생했을 때 처리하는 코드
        } catch (Exception2 e2) {
            // Exception2 예외가 발생했을 때 처리하는 코드
        } catch (Exception3 e3) {
            // Exception3 예외가 발생했을 때 처리하는 코드
        } catch (ExceptionA | ExceptionB e5) {
            // ExceptionA, ExceptionB 예외가 발생했을 때 처리하는 코드
        } finally {
            // 예외 발생 여부와 상관없이 항상 수행되어야 하는 코드
        }
        // 발생한 예외와 일치하는 catch 블록이 없으면 예외는 처리되지 않는다.
    }
}
```

## 예외 발생 정보

- printStackTrace() : 예외 발생 당시 호출 스택(Call Stack)에 있었던 메서드의 정보와 예외 메시지를 화면에 출력
- getMessage() : 발생한 예외 클래스의 인스턴스에 저장된 메시지를 얻어옴

```java
class Example {
    public static void main(String[] args) {
        try {
            int result = 3 / 0;
            System.out.println(result);
        } catch (ArithmeticException e) {
            e.printStackTrace(); // java.lang.ArithmeticException: / by zero, at Example.main(Example.java:5)
            System.out.println(e.getMessage()); // / by zero
        }
    }
}
```

## 예외 발생시키기

1. 발생시키려는 예외 클래스의 객체를 생성
2. 예외 객체를 `throw` 키워드를 이용하여 예외 발생

```java
class Example {
    public static void main(String[] args) {
        try {
            Exception e = new Exception("고의 발생");
            throw e;
        } catch (Exception e) {
            System.out.println("에러 메시지 : " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

## 메서드에 예외 선언

```java
class Example {
    void method() throws Exception1, Exception2, Exception3 {
        // 예외가 발생할 수 있는 코드
    }

    void method2() throws Exception { // Exception 클래스를 상속받은 모든 예외를 선언 -> 모든 예외를 처리할 수 있음
        // 예외가 발생할 수 있는 코드
    }
}
```

메서드 선언부에 예외를 선언함으로써 메서드를 사용하려는 사람에게 이에 대한 처리를 강요할 수 있으며 그와 동시에 어떤 예외가 발생할 수 있는지 알려줄 수 있다.

`public final void wait() throws InterruptedException`

예를 들어 `Object` 클래스의 `wait()` 메서드를 사용하려는 사람에게 이 메서드를 호출하고자 하는 메서드 내에 `InterruptedException` 처리를 해주어야 함을 알려줄 수 있다.  
추가적으로 `wait()` 에는 `InterruptedException` 이외에도 `IllegalMonitorStateException` 이 발생할 수 있는데, `IllegalMonitorStateException`
은 `RuntimeException`(`unchecked exception`) 을 상속받은 예외는 메서드 선언부에 예외를 선언하지(처리하지) 않아도 된다.

메서드의 throws 절에 선언된 예외는 예외를 처리하는 것이 아닌 호출한 메서드에 예외를 전달하는 것이다.  
예외를 계속 호출 스택에 있는 메서드로 전달하다가 main 메서드까지 처리되지 않으면 프로그램은 종료된다.

```java
class Exception {
    public static void main(String[] args) {
        try {
            File f = createFile(args[0]);
            System.out.println(f.getName() + "파일이 성공적으로 생성되었습니다.");
        } catch (Exception e) {
            System.out.println(e.getMessage() + " 다시 입력해 주시기 바랍니다.");
        }
    }

    static File createFile(String fileName) throws Exception {
        if (fileName == null || fileName.equals("")) {
            throw new Exception("파일이름이 유효하지 않습니다.");
        }
        File f = new File(fileName);
        f.createNewFile();
        return f;
    }
}
```

위 코드 처럼 `createFile`에서 예외가 발생했지만 직접 처리 하지 않고 호출한 `main` 메서드에서 예외를 처리할 수 있다.

## 사용자정의 예외

보통 `Exception` 혹은 `RuntimeException` 클래스를 상속받아 사용자 정의 클래스를 만들며, 필요에 따라 알맞는 클래스를 선택하면 된다.

```java
class MyException extends Exception {
    private final int ERR_CODE;

    MyException(String msg, int errCode) {
        super(msg);
        ERR_CODE = errCode;
    }

    MyException(String msg) {
        this(msg, 100);
    }

    public int getErrorCode() {
        return ERR_CODE;
    }
}
```

기존에는 위 예시처럼 `Exception` 클래스를 상속받아 사용자 정의 예외를 만들었지만, `RuntimeException` 클래스를 상속받아 사용자 정의 예외를 만들어 예외 처리를 강제하지 않는다.  
`checked exception`은 불필요한 경우에도 `try-catch`문을 넣어야 하기 때문에 코드가 복잡해지고 가독성이 떨어지는 단점이 있기 때문이다.

```java
class uncheckedException {
    public static void main(String[] args) {
        try {
            generateException(true, true);
        } catch (CheckedException e) {
            System.out.println("Caught checked exception");
        }
//        catch (UncheckedException e) {
//            System.out.println("Caught unchecked exception");
//        }
        // unchecked Exception 이므로 catch 블록 처리 하지 않아도 컴파일 에러가 발생하지 않는다.
    }

    static void generateException(boolean check, boolean uncheck) throws CheckedException /* , UncheckException */ { // unchecked exception 이므로 throws 절에 명시하지 않아도 컴파일 에러가 발생하지 않는다.
        if (check) {
            throw new CheckedException("CheckedException 발생");
        }

        if (uncheck) {
            throw new UncheckedException("UncheckException 발생");
        }
    }
}


class CheckedException extends Exception {
    CheckedException(String msg) {
        super(msg);
    }
}

class UncheckedException extends RuntimeException {
    UncheckedException(String msg) {
        super(msg);
    }
}
```

## 예외 되던지기

> 예외를 처리한 후에 인위적으로 다시 예외를 발생시키는 것

```java
class ExceptionEx3 {
    public static void main(String[] args) {
        try {
            method1();
        } catch (Exception e) {
            System.out.println("main 메서드에서 예외가 처리되었습니다.");
        }
    }

    static void method1() throws Exception { // method1 에서 발생한 예외를 main 메서드로 넘겨준다.
        try {
            throw new Exception();
        } catch (Exception e) {
            System.out.println("method1 메서드에서 예외가 처리되었습니다.");
            throw e; // method1 에서 발생한 예외를 다시 발생시킨다.
        }
    }
}
```

###### 참고자료

- [Java의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001)
- https://rollbar.com/blog/java-exceptions-hierarchy-explained/