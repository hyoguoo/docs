# Persistence Context(영속성 컨텍스트)

> 엔티티를 영구 저장하는 환경

## 엔티티의 생명 주기

1. 비영속(new/transient)
    - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
2. 영속(managed)
    - 영속성 컨텍스트에 관리되는 상태
3. 준영속(detached)
    - 영속성 컨텍스트에 저장되어있다가 분리된 상태
4. 삭제(removed)
    - 삭제된 상태

```java
class Example {
    public static void main(String[] args) {
        Member member = new Member(); // 비영속
        member.setId("member1");
        member.setUsername("회원1");

        EntityManager entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();
        entityManager.persist(member); // 영속

        entityManager.detach(member); // 준영속
        entityManager.remove(member); // 삭제
    }
}
```

## 영속성 컨텍스트의 이점

### 1차 캐시

`entityManager.persist(member)` 처럼 엔티티를 영속성 컨텍스트에 저장하면, 영속성 컨텍스트는 엔티티를 1차 캐시에 저장하고, 키를 통해 엔티티를 관리한다.

| @Id |            Entity            |
|:---:|:----------------------------:|
|  1  | Member(id=1, username="회원1") |

위 상태에서 `entityManager.find(Member.class, 1)` 을 호출하면, 데이터베이스에서 조회하기 전에 1차 캐시에서 조회하고, 없으면 데이터베이스에서 조회한다.  
마찬가지로 데이터베이스에 없어서 조회한 경우에도 그 결과를 1차 캐시에 저장한다.  
떄문에 캐시를 통해 조회하게 되는 경우 DB 조회 쿼리를 실행하지 않는다.

### 동일성(identity) 보장

```java
class Example {
    public static void main(String[] args) {
        Member member1 = entityManager.find(Member.class, 1L);
        Member member2 = entityManager.find(Member.class, 1L);
        System.out.println(member1 == member2); // true
    }
}
```

1차 캐시를 통해 조회하면, 동일한 트랜잭션에서 조회한 엔티티는 같은 인스턴스를 반환하기 때문에 동일한 인스턴스를 반환한다.

### 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)

```java
class Example {
    public static void main(String[] args) {
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();

        Member member1 = new Member(150L, "A");
        Member member2 = new Member(160L, "B");

        entityManager.persist(member1);
        entityManager.persist(member2);
        // 여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

        entityManager.getTransaction().commit(); // 여기서 INSERT SQL을 데이터베이스에 보낸다.
    }
}
```

`persist(*)`를 호출하면, 엔티티를 1차 캐시에 저장하고, 그와 동시에 INSERT SQL을 생성하고 바로 데이터베이스에 보내지 않고, 쓰기 지연 SQL 저장소에 저장하게 된다.  
그리고 트랜잭션을 커밋하면, 쓰기 지연 SQL 저장소에 저장된 쿼리를 데이터베이스에 보내게 된다.

### 변경 감지(Dirty Checking)

```java
class Example {
    public static void main(String[] args) {
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();

        Member member = entityManager.find(Member.class, 150L);
        member.setUsername("OGU");

        // entityManager.update(member); 와 같은 메서드가 존재하지 않고, persist()를 호출하지 않아도 된다.

        entityManager.getTransaction().commit(); // 여기서 UPDATE SQL을 데이터베이스에 보낸다.
    }
}
```

트랜잭션을 커밋하면, 1차 캐시에 있는 엔티티와 스냅샷을 비교해서 변경된 엔티티를 찾고, UPDATE SQL을 생성해서 데이터베이스에 보낸다.  
비슷하게 엔티티 삭제하는 경우에도, 1차 캐시에 있는 엔티티와 스냅샷을 비교해서 삭제 쿼리를 생성해서 데이터베이스에 보낸다.

## 플러시(Flush)

영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것으로, 아래와 같은 절차로 진행된다.

1. 변경 감지
2. 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
3. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송(등록, 수정, 삭제 쿼리)

기본적으로 트랜잭션 커밋을 호출하면 자동 호출되고, 아래의 상황에서 플러시가 호출된다.

1. 트랜잭션 커밋
2. JPQL 쿼리 실행
3. `entityManager.flush()` 메서드 직접 호출(거의 사용하지 않으나 테스트에서 사용할 수 있다.)

위에서 JPQL 쿼리 실행이 플러시를 호출하는 이유는, 아래와 같은 상황에서 엔티티가 조회되지 않는 문제를 방지하기 위해서이다.

```java
class Example {
    public static void main(String[] args) {
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();

        Member member = new Member(150L, "A");
        entityManager.persist(member);

        // 위에서 저장된 엔티티도 조회하기 위해 플러시를 호출하여 데이터베이스에 반영한 뒤 조회한다.
        List<Member> memberList = entityManager.createQuery("SELECT m FROM Member m", Member.class).getResultList();

        System.out.println("member = " + member);
    }
}
```

플러시를 직접 호출한다고 해서 영속성 컨텍스트를 비우는 것은 아니며, 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 것이다.

### 플러시 모드 옵션

- `FlushModeType.AUTO`: 커밋이나 쿼리를 실행할 때 플러시(기본값)
- `FlushModeType.COMMIT`: 커밋할 때만 플러시

## 준영속 상태

영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)된 상태를 말하며, 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.

### 준영속 상태로 만드는 방법

- `entityManager.detach(entity)`: 특정 엔티티만 준영속 상태로 전환
- `entityManager.clear()`: 영속성 컨텍스트를 완전히 초기화
- `entityManager.close()`: 영속성 컨텍스트를 종료

### 병합(merge)

```java
class Example {
    @Transactional
    void update(Item itemParam) { // itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
        Item mergeItem = em.merge(itemParam); // 준영속 상태의 엔티티를 영속 상태로 변경
    }
}
```

준영속 상태의 엔티티를 영속 상태로 변경하는 기능으로 동작 방식은 아래와 같다.

1. 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티 조회
    - 만약 1차 캐시에 없으면 데이터베이스에서 조회 후 1차 캐시에 저장
2. 조회한 영속 엔티티(mergeItem)에 준영속 엔티티(itemParam)의 값을 채워넣어 변경
3. 영속 엔티티(mergeItem)를 반환

여기서 준영속 엔티티의 값을 모두 교체하기 때문에 `null`로 업데이트 될 위험이 있어 사용하지 않는 것이 좋다.  
때문에 `merge()`를 통한 변경보다는 변경 감지 기능을 사용하여 변경하는 것이 좋다.

###### 참고자료

- [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic)