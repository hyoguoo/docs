---
layout: editorial
---

# CAS(Compare And Swap)

특정 메모리 위치의 값을 비교하고, 예상하는 값이 일치하면 새로운 값으로 교체하는 연산이다.

1. 인자로 기존 값, 새로운 값을 전달
2. 기존 값이 현재 메모리의 값과 같다면 새로운 값으로 교체
3. 교체 성공 여부를 반환

비교 + 교체 두 개의 연산으로 보이지만, 이는 하드웨어 수준에서 처리할 수 있어 원자적으로 동작하며, 동시에 여러 스레드가 동시에 CAS 연산을 수행해도 문제가 발생하지 않는다.

## Java에서의 CAS

`java.util.concurrent.atomic` 패키지에서 제공하는 `Atomic` 클래스들은 CAS 연산을 지원한다.  
CAS 연산은 Atomic 클래스의 메서드로 구현되어 있으며, 각 타입에 대응하는 클래스가 제공된다.(AtomicInteger, AtomicLong, AtomicBoolean 등)

```java
public class CASExample {

    public static void main(String[] args) {
        AtomicInteger atomicInt = new AtomicInteger(0);

        int expectedValue = 0;
        int newValue = 1;

        boolean success = atomicInt.compareAndSet(expectedValue, newValue);
        System.out.println("CAS 성공 여부: " + success);
        System.out.println("현재 값: " + atomicInt.get());
    }
}
```

### Atomic과 volatile

Atomic 클래스는 내부적으로 `volatile` 키워드를 사용하여 멤버 변수를 선언한다.

```java
public class AtomicInteger extends Number implements java.io.Serializable {

    private static final long serialVersionUID = 6214790243416807050L;

    // ...

    private volatile int value;

    // ...
}
```

volatile 키워드는 메모리 가시성을 보장하기 위해 사용되는데, 이는 CAS 연산이 올바르게 동작하기 위한 필수 조건이다.

- volatile 키워드를 사용하면, 변수의 값을 읽을 때 CPU 캐시가 아닌 메인 메모리에서 읽어오게 됨
- 멀티스레드 환경에서 모든 스레드가 항상 최신 값을 읽을 수 있도록 보장
- CAS 연산은 비교와 교체를 하는 동안 메모리 값이 정확해야 하므로, 메모리 가시성이 보장되어야 함

### vs 락 기반 동기화

여러 스레드가 동시에 하나의 자원에 접근할 때, 락 기반 동기화와 CAS 방식 두 가지 방법을 사용할 수 있다.

- CAS 방식: 간단한 연산과 충돌이 적은 환경에 적합 / 충돌 발생 시 재시도로 인해 성능 저하 가능
- 락 기반: 복잡한 동기화 로직이나 충돌이 빈번한 환경에 적합 / 락 획득 과정에서 경합과 대기 발생 가능

|     특징     |    락(Lock) 방식    | CAS(Compare-And-Swap) 방식 |
|:----------:|:----------------:|:------------------------:|
|   접근 방식    | 비관적(pessimistic) |     낙관적(optimistic)      |
|   동작 원리    |  락 획득 후 데이터 접근   |    값 비교 후 조건 만족 시 교체     |
| 복잡한 동기화 처리 |        적합        |           부적합            |
|  충돌 시 처리   |        대기        |           재시도            |
