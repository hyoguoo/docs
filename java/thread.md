---
layout: editorial
---

# Thread(스레드)

> 프로세스의 자원을 이용해 작업을 수행하는 실행 단위

## 스레드 구현

구현 방법으로는 아래 두 개의 방법이 있으며 큰 차이는 없으나 Thread 클래스를 상속 받으면 다른 클래스를 상속 받을 수 없기 떄문에 Runnable 방법을 권장한다.

- Thread 클래스를 상속받아 구현

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        // Do something
    }
}
```

- Runnable 인터페이스를 구현

```java
public class MyThread implements Runnable {

    @Override
    public void run() {
        // Do Something
    }
}
```

## 스레드 실행

생성된 스레드를 실행하기 위해서는 `run()`이 아닌 `start()` 메서드를 호출해야 한다.

```java
class ThreadEX1 extends Thread {

    public void run() {
        for (int i = 0; i < 10; i++) {
            // 조상 클래스에 구현 된 getName() 메서드를 이용해 스레드 이름을 출력
            System.out.println("ThreadEX1: " + i + "번째 실행" + " Thread Name: " + getName());
        }
    }
}

class ThreadEX2 implements Runnable {

    public void run() {
        for (int i = 0; i < 10; i++) {
            // Thread 클래스의 static 메서드인 currentThread()를 이용해 현재 실행 중인 스레드를 반환받아 getName() 메서드를 이용해 스레드 이름을 출력
            System.out.println("ThreadEX2: " + i + "번째 실행" + " Thread Name: " + Thread.currentThread().getName());
        }
    }
}

class ThreadMain {

    public static void main(String[] args) {
        ThreadEX1 threadEX1 = new ThreadEX1();

        // Runnable 인터페이스를 구현한 객체를 Thread 클래스 생성자의 매개변수로 전달
        Thread thread = new Thread(new ThreadEX2());

        threadEX1.start();
        thread.start();
    }
}
```

`start()` 메서드가 호출되어도 해당 스레드에서 바로 실행 되는 것이 아니기 때문에, 스레드 스케줄러에 의해 대기 상태가 되고 대기 상태에 있는 스레드는 스레드 스케줄러에 의해 실행된다.  
한 번 실행된 스레드는 다시 실행할 수 없으며 두 번 이상 실행하게 되면 `IllegalThreadStateException` 예외가 발생한다.

### `start()` vs `run()`

- `run()`: 생성된 스레드를 실행시키는 것이 아니라 단순히 클래스에 선언된 메서드를 호출
- `start()`: 새로운 스레드가 작업을 실행하는데 필요한 `call stack`을 생성하고 생성된 `call stack`에 `run()`메서드를 실행

호출 스택을 강제로 출력하는 코드 사용하여 결과를 확인해보면 `start()` 메서드를 호출한 경우 main 스레드와 별도의 스레드가 생성되어 실행되는 것을 확인할 수 있다.  
`run()`을 통해 실행하게 되면 단순히 메서드를 호출하는 것이기 때문에 main 스레드에서 `run()` 메서드가 실행되는 것을 확인할 수 있다.

```java
class ThreadEX1 extends Thread {

    public void run() {
        throwException();
    }

    private static void throwException() {
        try {
            throw new Exception();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class ThreadMain {

    public static void main(String[] args) {
        ThreadEX1 threadEX1 = new ThreadEX1();

        threadEX1.start();
    }
}

/*
java.lang.Exception
	at ThreadEX1.throwException(scratch.java:8)
	at ThreadEX1.run(scratch.java:3)
 */
```

```java
class ThreadEX1 extends Thread {

    public void run() {
        throwException();
    }

    private static void throwException() {
        try {
            throw new Exception();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class ThreadMain {

    public static void main(String[] args) {
        ThreadEX1 threadEX1 = new ThreadEX1();

        threadEX1.run();
    }
}

/*
java.lang.Exception
	at ThreadEX1.throwException(scratch.java:8)
	at ThreadEX1.run(scratch.java:3)
	at ThreadMain.main(scratch.java:19)
 */
```

모든 스레드는 작업을 수행하기 위해 자신만의 `call stack`이 필요한데, 새로운 스레드를 생성하고 실행하면 아래와 같은 순서로 실행된다.

1. 새로운 스레드가 생성되고 `run()` 메서드가 호출되어 새로운 `call stack`이 생성
2. `run()` 메서드가 종료되면서 해당 스택이 비워지게 되면서, 해당 스레드는 종료되고 `call stack`도 소멸

결국 main 스레드에서 `start()` 메서드를 호출하면 아래의 순서로 실행된다.

1. main 메서드에서 구현한 스레드의 `start()` 메서드를 호출
2. `start()`는 새로운 스레드를 생성하고, 스레드가 작업하는데 사용될 `call stack`을 생성
3. 생성된 스레드가 새로 생성된 `call stack`에 `run()` 메서드를 호출
4. 스케줄러에 의해 번갈아 가며 작업 수행

## 스레드 우선순위

스레드는 `priority`라는 속성(멤버변수)를 가지고 있으며 이 속성은 스레드의 우선순위를 나타낸다.

- 스레드의 우선순위는 1 ~ 10 사이의 값을 가질 수 있으며 기본값은 5
- OS마다 다른 방식으로 스케쥴링하기 때문에 실행 결과는 정확히 알 수 없음

## 실행제어

멀티 스레드 프로그래밍이 어려운 이유는 동기화(synchronization)와 스케줄링(scheduling) 때문이다.  
효율적인 멀티스레드 관리를 위해서는 우선 관련 메서드와 스레드의 상태를 이해해야 한다.

### 스케줄링 메서드

|                 메서드                  |                                    설명                                     |
|:------------------------------------:|:-------------------------------------------------------------------------:|
|   `static void sleep(long millis)`   |                            지정된 시간 동안 스레드 일시정지                             |
| `void join(), void join(long mills)` | 지정된 시간동안 스레드가 실행되도록 함, 지정된 시간이 지나거나 작업이 종료되면 `join()`을 호출한 스레드로 다시 돌아와 실행 |
|          `void interrupt()`          |            `sleep()`/`join()`에 의해 일시정지 상태인 스레드를 깨워 실행대기 상태로 만듬            |
|            `void stop()`             |                                스레드를 강제로 종료                                |
|            `void yield()`            |                  다른 스레드에게 실행 양보하고, 호출한 스레드는 실행대기 상태로 변경                   |
|           `void suspend()`           |               스레드를 일시정지 상태로 만듬, `resume()`을 호출할 때까지 실행되지 않음               |
|           `void resume()`            |               `suspend()`에 의해 일시정지 상태인 스레드를 다시 실행대기 상태로 만듬                |

** `resume()`, `stop()`, `suspend()`는 스레드를 교착상태(dead-lock)로 만들기 쉽기 때문에 deprecated 됨

### 스레드의 상태

|           상태            |                                         설명                                          |
|:-----------------------:|:-----------------------------------------------------------------------------------:|
|           NEW           |                         스레드가 생성되고 아직 `start()`가 호출되지 않은 상태                          |
|        RUNNABLE         |                                  실행 중 또는 실행 가능한 상태                                  |
|         BLOCKED         |                     동기화 블럭에 의해 일시정지된 상태(`lock`이 풀릴 때까지 기다리는 상태)                     |
| WAITING / TIMED_WAITING | 스레드의 작업이 종료되지는 않았지만 실행가능하지 않은(`unrunnable`) 일시정지 상태(`TIMED_WAITING`은 일시정지시간이 지정된 경우 |
|       TERMINATED        |                                   스레드의 작업이 종료된 상태                                   |

### 스레드 생성 - 소멸 과정

![Tread Lifecycle](image/thread-lifecycle.png)

1. 스레드 생성하고 `start()` 호출하여 실행 대기열에 저장되어 실행 대기 상태로 만듬
    - 실행대기열은 큐(queue)와 같은 구조로 먼저 실행대기열에 들어온 스레드가 먼저 실행됨
2. 실행 대기열에 있다가 차례가 되면 실행상태로 변경
3. 주어진 실행시간이 다되거나 `yield()`를 만나면 실행대기상태가 되고 다시 실행대기열에 들어감
4. 실행 중 `suspend()`, `sleep()`, `join()`, `wait()`, `I/O block` 의해 일시정지 상태가 될 수 있음
5. 지정된 일시정지시간이 다 되거나 `time-out()`, `notify()`, `resume()`, `interrupt()` 등의 메서드를 호출하면 다시 실행대기상태가 됨
6. 실행을 모두 마치거나 `stop()`을 호출하면 종료상태가 됨

###### 참고자료

- [Java의 정석](https://kobic.net/book/bookInfo/view.do?isbn=9788994492032)
