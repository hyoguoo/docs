# Lazy Loading / Eager Loading(지연 로딩 / 즉시 로딩)

## 지연 로딩(`FetchType.LAZY`)

엔티티 객체의 연관된 관계를 실제로 사용할 때까지 데이터베이스에서 로딩을 늦추는 것으로, 연관된 엔티티를 우선 프록시 객체로 조회하고, 실제로 사용할 때 데이터베이스에서 조회한다.

```java

@Entity
public class Post {
    @Id
    private Long id;

    private String title;

    @OneToMany(mappedBy = "post", fetch = FetchType.LAZY)
    private List<Comment> comments;
}

@Entity
public class Comment {
    @Id
    private Long id;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id")
    private Post post;
}
```

```java
public class Example {
    public static void main(String[] args) {
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();

        Post post = entityManager.find(Post.class, 1L); // post만 조회하고, comments는 조회하지 않고, 값이 없는 프록시 객체를 가지고 있음

        List<Comment> comments = post.getComments(); // comments를 사용할 때 데이터베이스에서 조회

        entityManager.getTransaction().commit();
        entityManager.close();
    }
}
```

## 즉시 로딩(`FetchType.EAGER`)

위에서 즉시 로딩으로 설정하게 되면, `entityManager.find(Post.class, 1L)`를 호출할 때 `comments`를 함께 조회하게 되면서 프록시 객체가 아닌 실제 객체를 조회하게 된다.  
즉시 로딩은 연관된 엔티티를 함께 조회하기 때문에 한 두개의 조인은 큰 문제가 되지 않지만, 복잡한 비즈니스 로직이 적용 된 테이블 연관 관계를 모두 조인하면 성능 이슈가 발생할 수 있다.  
때문에 모든 연관 관계를 지연 로딩으로 설정하고, 꼭 필요한 곳에서 `FetchType.EAGER`로 설정하는 것이 좋다.

- `***ToOne` 관계는 기본이 즉시로딩이므로 `fetch = FetchType.LAZY`로 설정
- `***ToMany` 관계는 기본이 지연로딩이므로 따로 설정할 필요 X

###### 참고자료

- [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic)