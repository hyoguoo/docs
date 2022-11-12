# 엔티티 설계

## 엔티티 설계 중 주의점

### 엔티티는 필요한 경우가 아니면 `Setter` 사용 X

변경 포인트가 많아지면 유지보수가 어려워진다.

### 모든 연관관계는 지연로딩(`lazy`)으로 설정

- 즉시로딩(`eager`)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다.
- `***ToOne` 관계는 기본이 즉시로딩이므로 `fetch = FetchType.LAZY`로 설정
- `***ToMany` 관계는 기본이 지연로딩이므로 따로 설정할 필요 X

### 컬렉션은 필드에서 초기화

- `null` 문제를 방지
- `Hibernate`는 엔티티를 영속화할 때, 컬렉션을 감싸서 `Hibernate`가 제공하는 내장 컬렉션으로 변경하는데, 임의의 메서드에서 컬렉션을 잘못 생성하면 내장 컬렉션으로 변경하지 못할 수 있다.

```java
class Example {
    public void example() {
        Member member = new Member();
        System.out.println(member.getOrders().getClass()); // java.util.ArrayList
        entityManager.persist(member);
        System.out.println(member.getOrders().getClass()); // org.hibernate.collection.internal.PersistentBag
    }
}
```

### 테이블, 컬럼명 생성 전략

- CamelCase -> SnakeCase
- `.` -> `_`
- 대문자 -> 소문자