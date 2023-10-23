---
layout: editorial
---

# Singleton(싱글톤)

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

## Java에서의 싱글톤 패턴

우선 Java에서 싱글톤 패턴을 구현하기 위해서는 다음과 같이 구현할 수 있다.

- 클래스의 인스턴스가 한 개만 생성되는 것을 보장하는 디자인 패턴
- 생성자를 private으로 선언하여 외부에서 new 키워드를 사용한 객체 생성을 막음

```java
public class SingletonService {

    // static 영역에 객체 instance를 하나만 생성
    private static final SingletonService instance = new SingletonService();

    // 생성자를 private으로 선언하여 외부에서 new 키워드를 사용한 객체 생성을 막음
    private SingletonService() {
    }

    // 해당 객체의 인스턴스가 필요할 경우 static 메서드인 getInstance()를 통해 이 instance를 공유
    public static SingletonService getInstance() {
        return instance;
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }

}
```

위 방법으로 순수 Java로 싱글톤 패턴을 구현할 수 있지만 여러 단점이 생겨나게 된다.

- 싱글톤 패턴을 구현하는 코드 자체가 늘어남
- 의존관계상 클라이언트가 구체 클래스에 의존 -> DIP 위반
- 클라이언트가 구체 클래스에 의존해서 OCP, 테스트, private 생성자로 자식 클래스 생성 불가 등의 문제점이 발생할 수 있음
- 테스트의 어려움
- 내부 속성을 변경하거나 초기화 하기 어려움
- private 생성자로 자식 클래스를 만들기 어려움
- 유연성이 떨어짐

## 싱글톤 레지스트리(Singleton Registry)

[스프링 컨테이너](container.md)에 객체들이 빈으로 등록되면서 객체를 생성하고 관리하는데, 이 때 싱글톤 레지스트리 패턴을 사용한다.  
때문에 스프링 컨테이너에 빈으로 등록되면 싱글톤으로 객체를 관리하게 되면서 위에 언급 된 문제점을 해결할 수 있다.

- 싱글톤 패턴을 위한 `getInstance()`와 같은 코드가 필요하지 않음
- DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있음

## 싱글톤 방식의 주의점

싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 무상태(stateless)로 설계해야 한다.

- 특정 클라이언트에 의존적인 필드 존재 X
- 특정 클라이언트가 값을 변경할 수 있는 필드 존재 X
- 가급적 읽기만 가능하도록 설계
- 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용

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

## @Configuration과 싱글톤

```java

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberService(memberRepository()); // memberRepository() 호출
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderService(memberRepository(), discountPolicy()); // memberRepository() 호출
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

위의 Config로 스프링 컨테이너에 등록되면 자바 코드 상으론 여러 번의 `memberRepository()` 호출을 하여 여러 개의 인스턴스를 생성하게된다.  
하지만 실제로는 한 번만 호출되며, 스프링이 CGLIB라는 라이브러리를 통해 @Configuration이 붙은 클래스를 프록시 클래스를 만들어 싱글톤을 보장한다.

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

Configuration 프록시 클래스가 스프링 빈으로 등록되어 다른 스프링 빈을 등록할 때, 이미 등록되어있는 스프링 빈은 그대로 반환하고 없으면 생성하여 반환하는 코드가 동적으로 만들어 싱글톤을 보장한다.

###### 참고자료

- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/스프링-핵심-원리-기본편)