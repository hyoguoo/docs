# 스프링 컨테이너

## 스프링 컨테이너

```java
public class Main {

    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    }
}
```

- `ApplicationContext`: 스프링 컨테이너라고 하며 인터페이스
- `AnnotationConfigApplicationContext`: 위 인터페이스로 구현한 것 중 하나로 `Anootation` 기반의 애플리케이션 컨텍스트 생성 구현체
- 스프링 컨테이너는 Annotation 기반 뿐만 아니라 XML 기반으로도 생성 가능

## 스프링 컨테이너 생성과정

1. 스프링 컨테이너 생성

- 스프링 컨테이너

| Bean Name | Bean Object |
|:---------:|:-----------:|
|     -     |      -      |
|     -     |      -      |
|     -     |      -      |
|     -     |      -      |

- `new AnnotationConfigApplicationContext(AppConfig.class)`로 config 정보를 넘겨주면서 구성 정보를 지정해준다.
- `AppConfig.class` 가 구성 정보로 해당

2. 스프링 빈 등록

- AppConfig

```java

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

- 스프링 컨테이너

|    Bean Name     |        Bean Object         |
|:----------------:|:--------------------------:|
|  memberService   |   MemberServiceImpl@x01    |
|   orderService   |    OrderServiceImpl@x02    |
| memberRepository | MemoryMemberRepository@x03 |
|  discountPolicy  |   RateDiscountPolicy@x04   |

- 스프링 컨테이너는 파라미터로 넘어온 클래스 정보를 사용해 스프링 빈 등록
- 빈 이름은 메서드 이름을 사용하며, `@Bean(name="memberServiceName")` 과 같이 다른 이름을 부여할 수 있다.
- 항상 다른 이름을 부여해야 하며, 중복 될 경우 버전에 따라 덮어 쓰거나 오류가 발생

3. 스프링 빈 의존관계 설정

```
memberService -> memberRepository <- orderService -> discountPolicy
```

- 스프링 컨테이너는 설정 정보를 참고해서 의존 관계를 주입(DI)

###### 출처

- https://www.inflearn.com/course/스프링-핵심-원리-기본편