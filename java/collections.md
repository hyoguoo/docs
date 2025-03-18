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

###### 참고자료

- [java T point](https://www.javatpoint.com/collections-in-java)
- [Oracle Docs](https://docs.oracle.com/javase/8/docs/api/)
