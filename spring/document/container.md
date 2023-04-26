# 스프링 컨테이너

스프링 컨테이너는 스프링에서 애플리케이션을 구성하는 여러 빈(Bean)들을 관리하는 공간을 말한다.  
스프링 컨테이너를 언급할 때는 빈 팩토리(`Bean Factory`)와 애플리케이션 컨텍스트(`Application Context`) 크게 두 가지로 나눌 수 있다.

## BeanFactory vs ApplicationContext

[interface]BeanFactory <-- [interface]ApplicationContext <-- [구현체]AnnotationConfigApplicationContext

### BeanFactory

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리하고 조회하는 역할
- `getBean()` 제공

### ApplicationContext

- `BeanFactory` 기능을 모두 상속 받아 제공
- 그 외에 아래를 포함한 많은 기능을 제공

|         interface         |         description          |
|:-------------------------:|:----------------------------:|
|       MessageSource       |            국제화 기능            |
|    EnvironmentCapable     |           환경변수 기능            |
| ApplicationEventPublisher |  이벤트를 발행하고 구독하는 모델을 편리하게 지원  |
|      ResourceLoader       | 파일/클래스패스/외부 등에서 리소스를 편리하게 조회 |

## 스프링 컨테이너 생성과정

1. 스프링 컨테이너 생성

- 스프링 컨테이너

| Bean Name | Bean Object |
|:---------:|:-----------:|
|     -     |      -      |
|     -     |      -      |
|     -     |      -      |
|     -     |      -      |

```java
public class Main {

    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    }
}
```

- `new AnnotationConfigApplicationContext(AppConfig.class)`로 config 정보를 넘겨주면서 구성 정보를 지정해준다.
- `AppConfig.class` 가 구성 정보에 해당

2. 스프링 빈 등록(`AppConfig`를 통한 예시)

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

###### 참고자료

- https://www.inflearn.com/course/스프링-핵심-원리-기본편