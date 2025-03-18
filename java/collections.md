---
layout: editorial
---

# Collections(컬렉션)

> 객체 그룹을 저장하고 조작하는 아키텍처를 제공하는 프레임워크

- 컬렉션 프레임워크 계층 구조

![Collection Tree Diagram](image/collections-tree.png)

## Collection Interface

Iterator 인터페이스를 상속한 가장 기본이 되는 인터페이스로 add, remove, contains, isEmpty, size, toArray 등의 메소드를 가지고 있다.

### Collections vs Collection

Collections는 인스턴스화할 수 없는 클래스이며, Collection 인터페이스를 구현한 클래스들을 다루는 static 메서드를 제공한다.

```java
public class Collections {

    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {
    }
}
```

[제공 메서드](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)

## Collection 하위 Class 특징

|     Class     |       Base Class       | Base Interface | Duplicate | Order | Sort |         Description         |
|:-------------:|:----------------------:|:--------------:|:---------:|:-----:|:----:|:---------------------------:|
|   ArrayList   |      AbstractList      |      List      |     O     |   O   |  X   |         배열 기반의 리스트          |
|    Vector     |      AbstractList      |      List      |     O     |   O   |  X   |    동기화를 지원하는 배열 기반의 리스트     |
|     Stack     |         Vector         |      List      |     O     |   O   |  X   | Vector의 하위 클래스로 LIFO 구조의 스택 |
|  LinkedList   | AbstractSequentialList |  List, Deque   |     O     |   O   |  X   |       연결 리스트 기반의 리스트        |
|  ArrayDeque   |   AbstractCollection   |     Deque      |     O     |   O   |  X   |          배열 기반의 덱           |
| PriorityQueue |     AbstractQueue      |     Queue      |     O     |   O   |  O   |           우선순위 큐            |
|    HashSet    |      AbstractSet       |      Set       |     X     |   X   |  X   |        해시 테이블 기반의 집합        |
| LinkedHashSet |        HashSet         |      Set       |     X     |   O   |  X   |    해시 테이블과 연결 리스트 기반의 집합    |
|    TreeSet    |      AbstractSet       |   SortedSet    |     X     |   O   |  O   |       이진 검색 트리 기반의 집합       |

## Vector & Stack 사용하지 않는 이유

Vector는 ArrayList와 같이 List 인터페이스를 구현한 컬렉션 프레임워크이며, 쓰레드 안전 여부를 제외하고는 ArrayList와 거의 동일하다.  
하지만 아래와 같은 이유로 현재 더 이상 사용하지 않는 것을 권장하고 있다.

### 1. `synchronized`로 인한 성능 저하

Stack과 Vector는 거의 모든 메서드에 `synchronized` 키워드가 붙어있어 멀티 쓰레드 환경에서 안전하게 사용할 수 있지만, 이로 인해 성능이 저하된다.  
동기화 처리라는 장점이 있다고 생각할 수 있지만, 메서드 레벨에서만 동기화 처리가 되어있어 여러 메서드를 호출하는 경우에는 여전히 안전하지 않다.

```java
public static void main(String[] args) {
    // ...
    if (!vector.isEmpty()) { // 이 시점에 다른 스레드가 remove를 수행할 수 있음
        vector.remove(0);
    }
}
```

** `ConcurrentHashMap`와 같은 동시성 컬렉션들은 내부적으로 CAS(Compare And Swap)를 사용하여 동기화 문제를 해결하고 있다.

### 후입선출 구조 위반

Stack에는 후입선출을 위한 메서드만 존재하지만, Vector는 List 인터페이스를 구현하고 있어 get, add, remove 등의 메서드를 사용할 수 있다.  
이는 Stack의 후입선출 구조를 위반하여 원하는 위치에 자유롭게 접근할 수 있게 된다.

```java
public static void main(String[] args) {
    Stack<Integer> stack = new Stack<>();
    stack.push(1);
    stack.push(2);
    stack.push(3);

    stack.add(1, 4); // Stack의 후입선출 구조를 위반
    stack.remove(1); // Stack의 후입선출 구조를 위반
}
```

###### 참고자료

- [java T point](https://www.javatpoint.com/collections-in-java)
- [Oracle Docs](https://docs.oracle.com/javase/8/docs/api/)
