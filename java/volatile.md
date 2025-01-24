---
layout: editorial
---

# volatile

코어에는 코어마다 별도의 캐시를 가지고 있는데, 읽어온 값을 캐시에 저장하고 캐시에서 값을 읽어온다.  
만약 다시 같은 값을 읽어오려고 하면 캐시에서 읽어오기 때문에 메모리의 값이 변경되어도 캐시에 저장된 값을 읽어올 수 있다.

```java
class Example {

    volatile boolean v = false;
}
```

`volatile`를 변수에 적용하면 캐시가 아닌 항상 메모리에서 직접 읽고 쓰도록 하여, 변수의 값이 변경되면 다른 스레드에서도 즉시 변경된 값을 읽을 수 있게 된다.

## 원자화

JVM은 데이터를 4byte 단위로 읽어오고 쓰기 때문에 `int`나 보다 작은 타입들은 원자적으로 읽고 쓰기가 가능하다.  
하지만 `long` 같은 큰 타입은 하나의 명령어로 읽고 쓸 수 없기 때문에 변수의 값을 읽고 쓰는 도중 다른 스레드가 값을 변경하면 값의 불일치가 발생할 수 있다.  
이는 `volatile`을 사용하거나 `synchronized`를 사용하여 해결할 수 있다.

- `volatile` : 해당 변수에 대한 읽기/쓰기를 원자적으로 처리
- `synchronized` : 해당 블록에 감싸진 코드를 원자적으로 처리

## volatile & synchronized

`volatile`는 변수의 읽기/쓰기만 원자화 시킬 뿐, 동기화 시키는 개념은 아니다.  
아래 코드에서 `balance`는 `volatile`로 선언되어 있지만, `getBalance()`와 `withdraw()`는 동기화 처리 되어 있지 않을 때,  
`balance`의 값이 변경되는 도중에 `getBalance()`가 호출되면 값의 불일치가 발생할 수 있다.

```java
class Example {

    volatile int balance;

    synchronized int getBalance() {
        return balance;
    }

    synchronized void withdraw(int amount) {
        balance -= amount;
    }
}
```

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
