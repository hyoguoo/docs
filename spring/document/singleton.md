# 싱글톤

웹 애플리케이션은 보통 여러 고객이 동시에 요청을 하게 되는데, 아래와 같이 코드를 짠다면 고객이 요청을 할 때마다 새 객체가 생성된다.

```java
public class SingletonTest {

    @Test
    @DisplayName("Pure DI Container")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();

        MemberService memberService1 = appConfig.memberService();
        MemberService memberService2 = appConfig.memberService();

        System.out.println("memberService1 = " + memberService1); // memberService1 = study.corebasic.member.MemberServiceImpl@35fc6dc4
        System.out.println("memberService2 = " + memberService2); // memberService2 = study.corebasic.member.MemberServiceImpl@7fe8ea47

        assertThat(memberService1).isNotSameAs(memberService2);
    }
}
```

그렇게 되면 인스턴스가 무수히 많이 생성하게 되어 메모리 낭비가 심해 성능 저하 이슈가 발생할 수 있다.  
-> 해당 객체를 딱 1개만 생성하고, 공유하도록 설계하는 싱글톤 패턴을 적용하여 해결할 수 있다.

## 싱글톤 패턴

- 클래스의 인스턴스가 한 개만 생성되는 것을 보장하는 디자인 패턴
- 생성자를 private으로 선언하여 외부에서 new 키워드를 사용한 객체 생성을 막음

```java
public class SingletonService {

    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance() {
        return instance;
    }

    private SingletonService() {
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }

}

```

- `static` 영역에 객체 `instance`를 하나만 생성
- 해당 객체의 인스턴스가 필요할 경우`static` 메서드인 `getInstance()`를 통해 이 `instance`를 공유
- 하나의 객체 인스턴스만 존재햐야하므로 생성자를 `private`으로 선언하여 외부에서 new 키워드를 사용한 객체 생성을 막음

### 위 방법으로 구현한 싱글톤 패턴의 문제점

- 싱글톤 패턴을 구현하는 코드 자체가 늘어남
- 의존관계상 클라이언트가 구체 클래스에 의존한다. -> DIP 위반
- 클라이언트가 구체 클래스에 의존해서 OCP, 테스트, private 생성자로 자식 클래스 생성 불가 등의 문제가 발생할 수 있다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스 생성하기 어렵다.
- 결론적으로 유연성이 떨어진다.
- 안티패턴으로 불리기도 한다.

## 싱글톤 컨테이너

위에서 언급 된 문제점을 해결하면서, 객체 인스턴스를 싱글톤으로 관리(스프링 빈도 싱글톤으로 관리되는 빈)

- 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서 객체 인스턴스를 싱글톤으로 관리
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 함
    - 싱글톤 레지스트리: 싱글톤 객체를 생성하고 관리하는 기능
- 위 기능(싱글톤 레지스트리)으로 실글톤 패턴의 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있음
    - 싱글톤 패턴을 위한 지저분한 코드를 들고 있지 않음
    - DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있음

## 싱글톤 방식의 주의점

- 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에
- 상태 `유지(stateful)` X, `무상태(stateless)` O
- 설계 시 주의점
    - 특정 클라이언트에 의존적인 필드 존재 X
    - 특정 클라이언트가 값을 변경할 수 있는 필드 존재 X
    - 가급적 읽기만 가능하도록 설계
    - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

### 문제 발생 예시

```java
public class StatefulService {

    private int price; // Stateful Field

    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        this.price = price; // Problem!
    }

    public int getPrice() {
        return price;
    }
}
```

```java
public class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        ConfigurableApplicationContext ac = new SpringApplicationBuilder(TestConfig.class)
                .run();

        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        statefulService1.order("userA", 10000);
        statefulService2.order("userB", 20000);

        int price = statefulService1.getPrice();
        System.out.println("price = " + price);

        assertThat(statefulService1.getPrice()).isEqualTo(10000); // fail
    }

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```

- 위 코드는 `statefulService1`과 `statefulService2`가 같은 `price` 공유
- `statefulService1`이 `price`를 변경하면 `statefulService2`도 변경되는 문제 발생

## @Configuration

```java

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository(); // 3번 호출 예상
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderService(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

- 자바 코드 상으로는 `memberRepository()`가 3번 호출 될 것 같지만 실제로는 1번만 호출

```shell
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.orderService
```

### 바이트코드 조작

스프링 컨테이너는 싱글톤 레지스트리로, 스프링 빈이 싱글톤이 되도록 보장해줘야하는데, 위의 자바 코드 상으로는 3번 호출되고 있다.  
그래서 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용해서 싱글톤을 보장해준다.

```java
public class ConfigurationSingletonTest {
    @Test
    void configurationDeep() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean = " + bean.getClass()); // bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$d7f7f2a2
    }
}
```

- 순수한 클래스 명인 `class hello.core.AppConfig`가 아닌 `class hello.core.AppConfig$$EnhancerBySpringCGLIB$$d7f7f2a2`로 출력
- 스프링이 `CGLIB`라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것
- `AppConfig@CGLIB`가 `@Bean`이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 스프링 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 생성하여
  싱글톤 패턴 보장

###### 출처

- https://www.inflearn.com/course/스프링-핵심-원리-기본편