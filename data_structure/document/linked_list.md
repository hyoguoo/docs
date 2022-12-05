# 연결 리스트

## 연결 리스트의 종류

### 단일 연결 리스트

리스트에 들어가는 각 원소는 아래와 같은 구성을 가지고 있다.

- value
- 다음 원소에 대한 연결고리

단일 연결 리스트의 첫 번째 원소는 리스트의 머리(head)라고 하며, 마지막 원소는 리스트의 꼬리(tail)라고 부른다.  
단일 연결 리스트는 다음 노드를 가리키는 포인터나 레퍼런스로만 구성되어 있기 떄문에 한 방향으로만 탐새할 수 있다.  
따라서 리스트를 탐색하기 위해서는 항상 첫 번째 원소(head)부터 시작해야하며, `Java`에서는 다음과 같이 구현될 수 있다.

```java
class ListElement<T> {
    public ListElement(T value) {
        data = value;
    }

    private T data;
    public ListElement<T> next;
}
```

### 이중 연결 리스트

단일 연결 리스트의 여러가지 단점을 극복할 수 있는 리스트로, 다음 노드를 가리키는 포인터나 레퍼런스만 있는 것이 아닌 이전 노드를 가리키는 포인터나 레퍼런스가 존재한다.  
따라서 어떤 원소에서 시작하든 리스트 전체를 탐색할 수 있게됐으며, `Java`에서는 다음과 같이 구현될 수 있다.

```java
class ListElement<T> {
    public ListElement(T value) {
        data = value;
    }

    private T data;
    public ListElement<T> next;
    public ListElement<T> previous;
}
```

### 원형 연결 리스트

단일 연결 리스트, 이중 연결 리스트로 구현할 수 있다. 원형 연결 리스트는 모든 원소에서 다음 원소를 가리키는 포인터나 레퍼런스가 반드시 존재해야한다.  
만약 리스트에 하나만 있다면 자기 자신을 가리키면 된다.

## 기초적인 연결 리스트 연산

### 첫 번째 원소(head) 추적

단일 연결 리스트에서는 head를 추적할 수 있어야한다. 그렇지 않을 경우엔 언어에 따라 가비지 컬렉터에 의해 제거되거나 길을 잃은 쓰레기 값이 되고 만다.  
따라서 원소를 head 앞에 추가하거나 제거 할 때는 head 포인터 혹은 레퍼런스를 갱신해주어야한다.

```java
class Main {
    public ListElement<Integer> insertInFront(ListElement<Integer> list, int data) {
        ListElement<Integer> newList = new ListElement<Integer>(data);  // data가 담긴 node 생성
        newList.setNext(list);  // 새로 생성 된 노드 뒤에 기존 리스트 연결
        return newList;
    }
}
```

### 리스트 종주

```java
class Main {
    public ListElement<Integer> findData(ListElement<Integer> head, int data) {
        ListElement<Integer> elem = head;
        while (elem != null && elem.value() != data) { // 리스트의 끝이거나 데이터가 일치할 때까지 탐색
            elem = elem.next();
        }
        return elem;
    }
}
```

### 원소의 삽입 및 삭제

단일 연결 리스트에서의 삭제할 경우에는 그 앞 원소의 연결 고리를 수정해주어야한다.

- 첫 번째 원소 삭제

```java
class Main {
    public void delete(ListElement<Integer> head, int data) {
        ListElement<Integer> nextElem = head.next;
        head.data = null;
        head.next = null;

        head = nextElem;
    }
}
```

- 중간 원소 삭제

```java
class Main {
    public void delete(ListElement<Integer> head, int data) {
        ListElement<Integer> elem = head;
        while (elem.next() != null) {
            if (elem.next.data == data) {
                elem.next = elem.next.next;
            } else {
                elem = elem.next;
            }
        }
    }
}
```