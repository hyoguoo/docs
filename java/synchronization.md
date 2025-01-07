---
layout: editorial
---

# 스레드 동기화(Synchronization)

스레드가 여러개인 경우에는 프로세스 자원을 공유하게 되는데, 이 때 여러 스레드가 동시에 하나의 자원을 사용하려고 할 때 문제가 발생할 수 있다.  
이러한 일을 방지하기 위해 다른 스레드의 접근을 막아주는 임계 영역(`critical section`)과 잠금(`lock`) 개념이 필요하다.

## `synchronized` 키워드

`synchronized` 키워드는 메서드나 블럭에 사용하게 되면, 아래와 같은 효과를 가진다.

1. 해당 코드 영역을 임계 영역으로 지정됨
2. 공유 데이터(객체)가 가지고 있는 잠금을 획득한 단 하나의 스레드만 이 영역 내의 코드를 수행할 수 있게 함
3. A 스레드가 임계 영역 내의 코드 수행
4. 코드 수행 중 B 스레드가 해당 인스턴스에 접근하게 되면, 락을 획득할 때까지 `BLOCKED` 상태로 대기
5. A 스레드가 모든 코드를 수행하고 락을 반납
6. B 스레드가 락을 획득하여 `RUNNABLE` 상태가 되고, 코드를 수행

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
- 두 방법 모두 임계 영역에 들어가는 순간 잠금을 획득하고, 임계 영역을 빠져나오는 순간 잠금을 반납
- 해당 영역을 잘 설정해야 멀티스레드 프로그램의 성능 저하를 방지할 수 있음
- `synchronized`는 특정 시간까지 대기하는 타임아웃이나, 인터럽트를 걸 수 없음
- 만약 락이 반납되지 않으면, 다른 스레드가 무한정 대기하게 되어 데드락이 발생할 수 있음

### 스레드 동기화 예시

```java
class Example {

    public static void main(String[] args) {
        Runnable runnable = new Runnable();
        new Thread(runnable).start();
        new Thread(runnable).start();
    }
}

class Account {

    private int balance = 1000;

    public int getBalance() {
        return balance;
    }

    public /*synchronized*/ void withdraw(int money) {
        if (balance >= money) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException ignored) {
            }
            balance -= money;
        }
    }
}

class Withdraw implements Runnable {

    Account acc = new Account();

    @Override
    public void run() {
        while (acc.getBalance() > 0) {
            int money = (int) (Math.random() * 3 + 1) * 100;
            acc.withdraw(money);
            System.out.println("balance : " + acc.getBalance());
        }
    }
}
```

위의 예시에서 `withdraw()` 메서드에 `synchronized` 키워드를 붙이지 않으면 잔액이 마이너스가 되는 경우가 발생한다.

- `withdraw()` 메서드가 임계 영역으로 지정되지 않으면 두 개 이상의 스레드가 동시에 `withdraw()` 메서드가 실행됨
- 여러 개의 스레드가 동시에 `balance` 변수의 값을 읽어오고, 비교 및 감소 연산을 수행하게 됨

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
