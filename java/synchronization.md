---
layout: editorial
---

# 스레드 동기화(Synchronization)

스레드가 여러개인 경우에는 프로세스 자원을 공유하게 되는데, 이 때 여러 스레드가 동시에 하나의 자원을 사용하려고 할 때 문제가 발생할 수 있다.  
이러한 일을 방지하기 위해 다른 스레드의 접근을 막아주는 임계 영역(Critical Section)과 잠금(Lock) 개념이 필요하다.

## `synchronized` 키워드

`synchronized` 키워드는 메서드나 블럭에 사용되며, 아래와 같은 효과를 제공한다.

1. 해당 코드 영역을 임계 영역으로 지정
2. 공유 데이터(객체)가 가지고 있는 잠금을 획득한 단 하나의 스레드만 이 영역 내의 코드를 수행할 수 있게 함
3. 임계 영역을 점유한 A 스레드가 코드 실행 중인 경우, B 스레드는 잠금 해제될 때까지 `BLOCKED` 상태로 대기
4. A 스레드가 임계 영역 실행 완료 후 락을 반납하면, B 스레드가 락을 획득하여 `RUNNABLE` 상태가 되고 코드를 수행

### `synchronized` 사용 방법

```java
class Example {

    // 1. 메서드 전체를 임계 영역 지정
    public synchronized void method() {
        // ...
    }

    public void method() {
        // 2. 메서드 내의 특정 영역을 임계 영역 지정
        synchronized (this) {
            // ...
        }
    }
}
```

- `synchronized` 키워드를 사용하면 해당 메서드나 블럭이 실행되는 동안에는 다른 스레드가 해당 메서드나 블럭에 접근할 수 없음
- 임계 영역에 들어가는 순간 잠금을 획득하고, 임계 영역을 빠져나오는 순간 잠금을 반납
- 해당 영역을 최소한의 범위로 설정해야 멀티스레드 프로그램의 성능 저하를 최소화할 수 있음

### 스레드 동기화 예시

```java
class Account {

    private int balance = 1000;

    public synchronized void withdraw(int money) {
        if (balance >= money) {
            try {
                Thread.sleep(100); // 지연을 추가하여 스레드 충돌 상황 테스트
            } catch (InterruptedException ignored) {
            }
            balance -= money;
            System.out.println(Thread.currentThread().getName()
                    + " withdrew " + money);
        } else {
            System.out.println(Thread.currentThread().getName()
                    + " attempted to withdraw " + money + " but insufficient balance.");
        }
    }

    public int getBalance() {
        return balance;
    }
}

class Withdraw implements Runnable {

    Account acc = new Account();

    @Override
    public void run() {
        while (acc.getBalance() > 0) {
            int money = (int) (Math.random() * 3 + 1) * 100;
            acc.withdraw(money);
            System.out.println("Remaining balance: " + acc.getBalance());
        }
    }
}
```

`withdraw()` 메서드가 `synchronized` 키워드를 붙여 동기화된 덕분에 잔액이 부족한 상황이 발생하지 않는다.

- 한 번에 하나의 스레드만 메서드에 진입하므로, balance가 0보다 작아지는 상황이 발생하지 않음
- 잔약이 부족한 경우에도 동기화로 인해 항상 정확한 잔액 조회하여 출금 가능 여부를 판단할 수 있음


- `withdraw()` 메서드가 임계 영역으로 지정되지 않으면 두 개 이상의 스레드가 동시에 `withdraw()` 메서드가 실행됨
- 여러 개의 스레드가 동시에 `balance` 변수의 값을 읽어오고, 비교 및 감소 연산을 수행하게 됨

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
