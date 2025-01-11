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

### `synchronized` 사용 시 주의사항

`synchronized`는 임계 영역을 손쉽게 지정할 수 있는 장점이 있지만, 타임아웃 설정이나 인터럽트 처리가 불가능하다.  
때문에 특정 조건에서 스레드가 락 획득을 위해 무한 대기 상태에 빠질 수 있다는 문제가 있다.

## `wait()`와 `notify()`를 활용한 동기화

`wait()`와 `notify()`는 스레드 간의 통신을 위해 사용되는 `Object` 클래스에서 제공되는 메서드로,  
`synchronized`와 함께 사용하여 스레드 간 락을 획득하고 반납하는 것을 조절할 수 있다.

- `wait()`
    - 현재 스레드가 가진 객체의 락을 반납하고 `WAITING` 상태로 전환하여 스레드 대기 집합으로 이동
- `notify()`
    - 스레드 대기 집합에 있는 스레드 중 하나를 깨움(여러 스레드가 대기 중인 경우, 어떤 스레드가 깨어날지는 알 수 없음)

### `synchronized`와 `wait()`/`notify()` 동작 과정

임계 영역을 다루기 위해 자바 내부에서는 아래 세 가지 기본 요소를 가진다.

- 모니터 락(Monitor Lock): 모든 객체는 모니터 락을 가지고 있으며, `synchronized` 키워드로 임계 영역을 지정하면 해당 객체의 모니터 락을 사용
- 락 대기 집합(Wait Set): 락을 획득하려는 스레드가 이미 사용 중인 경우, 락 대기 집합에 스레드를 추가하여 대기
- 스레드 대기 집합(Thread Wait Set): `wait()` 메서드를 호출한 스레드가 대기하는 집합

동작 과정은 아래와 같다.

1. `synchornized` 임계 영역에 들어가기 위해 모니터 락 획득 필요
2. 이미 사용 중인 경우 락 대기 집합에서 `BLOCKED` 상태로 대기
3. 락이 반납되면 락 대기 집합에 있는 스레드 중 하나가 락을 획득하고 `RUNNABLE` 상태로 전환
4. `wait()`를 호출하면 스레드 대기 집합으로 이동하고 락을 다시 반납
5. 다른 스레드가 `notify()`를 호출하면 스레드 대기 집합에 있는 대기 중인 스레드 중 하나를 깨움
6. 깨어난 스레드는 `BLOCKED` 상태로 전환하여 락 대기 집합으로 이동
7. 락 획득을 시도하고 성공하면 `RUNNABLE` 상태로 전환
8. `wait()`를 호출한 부분부터 다시 실행

### `wait()`와 `notify()` 사용 예시

```java

class Account {

    private int balance = 0;

    // synchronized 메서드로 입금 처리
    public synchronized void deposit(int amount) {
        balance += amount;
        System.out.println(Thread.currentThread().getName() + " deposited " + amount);
        notify(); // 입금 후 대기 중인 출금 스레드를 깨움
    }

    // synchronized 메서드로 출금 처리
    public synchronized void withdraw(int amount) {
        while (balance < amount) {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting to withdraw " + amount);
                wait(); // 잔액 부족 시 대기
            } catch (InterruptedException ignored) {
            }
        }
        balance -= amount;
        System.out.println(Thread.currentThread().getName() + " withdrew " + amount);
    }

    public synchronized int getBalance() {
        return balance;
    }
}
```

## ReentrantLock

`ReentrantLock`은 `synchronized` 키워드와 유사하게 임계 영역을 지정할 수 있으며, 더 유연한 동기화 처리를 제공한다.  
기존 `synchronized`와는 다르게 모니터 락이 아닌 직접적인 락 객체를 사용하는 방식으로 동작한다.

1. 락 획득 타임아웃 설정 가능
    - `tryLock(long timeout, TimeUnit unit)` 메서드를 사용하여 락 획득 시도 시간 설정 가능
    - 락 획득 실패 시 다른 로직을 수행하거나 재시도를 하여 데드락을 방지할 수 있음
2. 인터럽트 처리 가능
    - `lockInterruptibly()` 메서드를 사용하여 락 대기 중에 인터럽트 처리 가능
3. 세분화된 락 제어
    - 여러 `Condition` 객체를 생성하여 세부적인 대기/알림 제어를 구현 가능

|    **특징**    |        **`synchronized`**         |     **`ReentrantLock`**      |
|:------------:|:---------------------------------:|:----------------------------:|
|   **락 타입**   |           모니터 락(JVM 관리)           |       객체 기반 락 (직접 관리)        |
| **타임아웃 지원**  |                미지원                |        지원 (`tryLock`)        |
| **인터럽트 지원**  |                미지원                |   지원 (`lockInterruptibly`)   |
|  **락 세분화**   |                불가능                |  가능 (다중 `ReentrantLock` 사용)  |
| **대기/알림 제어** | `wait()` / `notify()` 메서드로 제한적 제어 | `Condition` 객체를 통한 세부적 제어 가능 |

### `ReentrantLock` 동작 방식

`syncronized` 키워드와 유사하게 두 가지 대기 상태를 관리한다.

- `ReentrantLock` 객체 대기 큐: 락을 획득하려는 스레드 대기 공간
- `Condition` 객체 스레드 대기 공간: `await()` 메서드에 의해 대기 중인 스레드 대기 공간

동작 과정은 아래와 같다.

1. 스레드에서 `lock()` 메서드를 호출하여 락을 획득하려 시도
2. 이미 사용 중인 경우 대기 큐에서 `WAITING` 상태로 전환하여 대기
3. `unlock()` 메서드를 호출하여 락을 반납하면 대기 큐에서 대기 중인 스레드 중 하나가 락을 획득하고 `RUNNABLE` 상태로 전환
4. `await()` 메서드를 호출하면 `WAITING` 상태로 전환하여 스레드 대기 공간으로 이동하고 락을 반납
5. 다른 스레드에서 `signal()` 메서드를 호출하면 스레드 대기 공간에 있는 대기 중인 스레드 중 하나를 깨움
6. 깨어난 스레드는 `WAITING` 상태로 전환하여 대기 큐로 이동
7. 락 획득을 시도하고 성공하면 `RUNNABLE` 상태로 전환
8. `await()`를 호출한 부분부터 다시 실행

### `ReentrantLock` 사용 예시

```java

class PrinterQueue {

    private final Lock lock = new ReentrantLock();
    private final Condition notFullCondition = lock.newCondition();
    private final Condition notEmptyCondition = lock.newCondition();

    private final Queue<String> queue = new LinkedList<>();
    private final int maxCapacity;

    public PrinterQueue(int maxCapacity) {
        this.maxCapacity = maxCapacity;
    }

    // 프린트 작업 추가
    public void addPrintJob(String job) {
        lock.lock();
        try {
            while (queue.size() == maxCapacity) {
                System.out.println(Thread.currentThread().getName() + " is waiting to add print job: " + job);
                notFullCondition.await(); // 큐가 가득 찬 경우 대기
            }
            queue.offer(job);
            System.out.println(Thread.currentThread().getName() + " added print job: " + job);
            notEmptyCondition.signal(); // 큐에 데이터가 추가되었으므로 소비자 알림
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println(Thread.currentThread().getName() + " was interrupted while adding a print job.");
        } finally {
            lock.unlock();
        }
    }

    // 프린트 작업 처리
    public void processPrintJob() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                System.out.println(Thread.currentThread().getName() + " is waiting for a print job to process.");
                notEmptyCondition.await(); // 큐가 비어 있는 경우 대기
            }
            String job = queue.poll();
            System.out.println(Thread.currentThread().getName() + " is processing print job: " + job);
            notFullCondition.signal(); // 큐에 공간이 생겼으므로 생산자 알림
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println(Thread.currentThread().getName() + " was interrupted while processing a print job.");
        } finally {
            lock.unlock();
        }
    }
}
```

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
