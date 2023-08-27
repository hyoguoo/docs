---
layout: editorial
---

# 의존성 주입(DI)

> 객체 간 의존성을 자신이 아닌 외부에서 두 객체 간의 관계를 결정해주는 디자인 패턴

올바른 방법으로 의존성을 주입했을 경우 아래의 장점을 얻을 수 있다.

- Test 용이
- 코드 재사용성 증가
- 객체 간 의존성 감소
- 객체 간 결합도를 낮춰 유연한 코드 작성

## 의존성 주입이 필요한 이유

- 기존 코드

```java
public class MessageSender {
    private EmailService emailService; // 구체적인 구현체에 의존

    public MessageSender() {
        this.emailService = new EmailService(); // 구체적인 구현체 생성
    }

    public void sendMessage(String message) {
        emailService.sendMessage(message); // 구체적인 구현체의 메서드 호출
    }
}
```

위 예시에서는 `MessageSender` 클래스가 `EmailService` 클래스에 의존하기 때문에 EmailService 클래스의 구현체가 변경되면 MessageSender 클래스도 변경되어야 한다.  
결국 객체들 간의 관계가 아닌 클래스 간의 관계가 형성되어 유연한 코드 작성이 어렵게 된다.

- 의존성 주입을 이용한 코드

```java
public interface MessageService {
    void sendMessage(String message);
}

public class EmailService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending email: " + message);
        // 이메일을 전송하는 코드
    }
}

public class MessageSender {
    private MessageService messageService; // 인터페이스에 의존

    public MessageSender(MessageService messageService) {
        this.messageService = messageService; // 의존성 주입
    }

    public void sendMessage(String message) {
        messageService.sendMessage(message); // 인터페이스의 메서드 호출
    }
}
```

위 예시에서는 `MessageSender` 클래스가 `MessageService` 인터페이스(역할)에 의존하게 된다.  
이렇게 되면 `MessageService` 인터페이스를 구현한 클래스가 변경되어도 `MessageSender` 클래스는 변경하지 않아도 된다.

## DI 3가지 방법(Spring)

스프링 프레임워크에서는 여러 방법으로 의존성을 주입할 수 있으며, 그 중에서 생성자 주입을 가장 많이 사용한다.

### 1. 생성자 주입

생성자를 통해서 의존 관계를 주입 받는 방법

```java

@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired // 생략가능
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

- 생성자 호출 시점에 딱 한 번만 호출되는 것이 보장
- 주입 받은 객체가 불변하며 필수적인 관계에 사용
- 만약 생성자가 하나라면 `@Autowired` 생략 가능

### 2. Field 주입

필드에 바로 주입하는 방법으로, 과거에 많이 사용되었으나 최근에는 권장하지 않는 방법이다.

```java

@Controller
public class MemberController {
    @Autowired
    private MemberService memberService;
}
```

- 주입한 필드를 변경할 수 없어 외부에서 접근이 불가능하기 때문에 테스트 코드 작성에 어려움이 있음
- DI 프레임워크가 존재해야 동작하기 때문에 순수한 자바 코드로 테스트가 어려움
- 애플리케이션 실제 코드와 관계 없는 테스트 코드나 설정을 목적으로 하는 `@Configuration` 같은 곳에서만 특별한 용도로 사용하는것을 권장

### 3. 수정자 주입(Setter Injection)

Setter 메서드에 `@Autowired` Annotation을 붙이는 방법으로, `Setter` 메서드를 통해 의존성을 주입하는 방법이다.

```java

@Controller
public class MemberController {
    private MemberService memberService;

    @Autowired
    public void setMemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

- 선택/변경 가능성이 있는 의존관계에서 사용
- 결국 `Setter`를 `public`으로 열어두어야 하기 때문에 실행 중에 의존 관계가 변경 될 가능성 존재하여 위험

### 4. 일반 메서드 주입

일반 메서드를 통해 주입하는 방법으로, 거의 사용되지 않는 방법이다.

```java

@Controller
public class MemberController {
    private MemberService memberService;

    @Autowired
    public void init(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

- 한 번에 여러 필드를 주입 받을 수 있음
- 일반적으로 잘 사용하지 않음

## 생성자 주입을 사용하는 이유

기본적으로 `Spring Framework`에서 권장하는 방법은 1번의 `생성자 주입`이며, 그 이유는 다음과 같다.

- 대부분 의존관계 주입은 한 번 일어나면 애플리케이션 종료 시점까지 변경할 일이 없으며 변하면 안 된다(불변)
- 3번의 `Setter`방식은 실행 중 의존성이 변경되는 위험한 가능성이 있다.

### 1. 순환 참조 방지

개발 단위가 커지다 보면 여러 `Component` 간에 의존성이 생기며 A <--> B 컴포넌트가 아래와 같이 서로 참조하는 코드를 작성하게 되는 경우도 생기게 된다.

```java

@Service
public class AService {
    @Autowired
    private BService bService;

    public void HelloWorldA() {
        bService.HelloWorldB();
    }
}

@Service
public class BService {
    @Autowired
    private AService bService;

    public void HelloWorldB() {
        bService.HelloWorldA();
    }
}
```

위와 같이 됐을 때, 계속해서 객체를 서로 참조하는 메서드가 실행되어 무한루프가 되어버리기 때문에 StackOverflow 에러가 발생한다.  
생성자 주입도 마찬가지로 순환 참조가 발생하게 되지만 에러 발생 시점이 다르다.

- 생성자 주입: 애플리케이션을 실행하는 시점에 에러 발생
- 필드 주입 및 수정자 주입: 애플리케이션 실행 중 에러 발생(스프링 부트 2.6 이상부터는 생성자 주입과 마찬가지로 애플리케이션 실행 시점에 에러 발생)

### 2. 불변성

생성자로 의존성을 주입할 경우 `final`로 선언할 수 있어, 런타임 도중에 의존성을 주입 받는 객체의 불변성을 보장해준다.(OOP 5가지 원칙 중, 개방-폐쇄 원칙)

### 3. 테스트 용이

테스트가 특정 프레임워크(Spring)에 의존하는 것보다는 순수 자바 코드로 테스트를 작성하는 것이 가장 좋다.  
생성자 주입이 아닌 다른 주입 방법을 사용했을 경우 순수 자바 코드로 Unit 테스트를 작성하는 것이 어려워진다.

- Service Code

```java

@Service
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    public void save(int id) {
        memberRepository.save(id);
    }
}
```

- Test Code

```java
public class MemberTest {

    @Test
    public void test() {
        MemberService memberService = new MemberService();
        memberService.save(59);
    }
}
```

위와 같이 필드를 통한 의존성 주입을 하게 되면 Spring 위에서 동작하지 않는 테스트 코드에선 의존 관계 주입이 되지 않는다.  
테스트 코드의 `memberService는` `null`이 되어 정상적인 테스트가 불가능해진다.  
만약 Setter 주입을 사용하면 테스트 코드에서 `setter`를 통해 의존성을 주입해줄 수 있지만, 불변성 측면에서 위배된다.

## Lombok

생성자 주입은 많은 장점을 가져다주지만 필드 주입에 비해 편의성 면에선 조금 떨어지는데 Lombok을 사용하면 이를 극복할 수 있다.  
생성자를 한 개만 두고 `@Autowired`를 할 수 있다면 Lombok 라이브러리의 `@RequiredArgsConstructor`를 사용하게 되어 생성자 관련 코드를 간략하게 사용할 수 있다.

```java
// before
@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}

// after
@Controller
@RequiredArgsConstructor
public class MemberController {
    private final MemberService memberService;
}
```

## 두 개 이상의 빈

`DiscountPolicy`를 구현한 두 개의 클래스가 있고, 두 클래스 전부 스프링 빈으로 등록하여 다른 클래스에 자동 주입했을 경우 `NoUniqueBeanDefinitionException`이 발생한다.

```java

@Component
public class FixDiscountPolicy implements DiscountPolicy {
}

@Component
public class RateDiscountPolicy implements DiscountPolicy {
}

// 의존성 주입
@Component
public class OrderService {

    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

하위 타입으로 변경 지정할 수 있지만 이는 DIP 위배하고, 유연성이 떨어지는 문제가 있어 다른 해결 방법을 사용하는 것이 좋다.

### 1. @Autowired 필드명 매칭

`@Autowired`는 타입 매칭을 시도하고, 같은 타입의 빈이 있으면 필드 이름/파라미터 이름으로 구분하여 주입하게 된다.

```java
public class OrderService {

    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(DiscountPolicy rateDiscountPolicy) { // 이름 변경
        this.discountPolicy = rateDiscountPolicy;
    }
}
```

### 2. Qualifier 사용

빈 이름 자체를 변경하는 개념은 아니고 추가 구분자를 붙여주어 주입 받는 곳에서 구분된 이름으로 주입받을 수 있게 해준다.

```java

@Component
@Qualifier("mainDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {
}

@Component
@Qualifier("fixDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
}

// 의존성 주입
@Component
public class OrderService {

    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

만약 `@Qualifier`에 해당 하는 `Qualifier`값이 없는 경우 그 이름의 스프링 빈을 추가로 찾게된다.(이런 상황이 발생하지 않는 것이 좋다.)

### 3. @Primary 사용

주입에 우선 순위를 정해주는 방법으로 여러 빈이 매칭 됐을 경우 해당 애노테이션을 가진 빈이 우선권을 가진다.

```java

@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {
}
```

`@Qualifier`는 사용하는 모든 코드에 붙여줘야 하지만, `@Primary`는 해당 코드에만 붙여주면 의존성 주입 정리가 끝난다.

### 정리

메인으로 사용 할 스프링 빈은 `@Primary` 를 적용해서 편리하게 조회하고, 서브로 사용할 빈은 `@Qualifier` 를 지정해서 명시적으로 획득 하는 방식으로 사용하면 코드를 깔끔하게 유지할 수 있다.

- `@Autowired`
    1. 타입 매칭
    2. 타입 매칭의 결과가 2개 이상일 때 필드명/파라미터명으로 빈 이름 매칭
- `@Qualifier`
    1. `@Qualifier`끼리 매칭
    2. 빈 이름 매칭
    3. 둘 다 없는 경우 `NoSuchBeanDefinitionException`
- `@Primary`
    1. 우선 순위 가짐

## 결론

> 생성자 자동 주입을 기본으로 사용

설정 정보 기반(`@Configuration`)으로 구성 부분/실제 동작 부분을 명확하게 나누는 것이 이상적이지만,  
빈을 하나 등록할 때 마다 설정 정보를 관리하는 것은 부담도 많이 크고, 자동으로 등록하더라도 `OCP`, `DIP`를 모두 보장 받을 수 있다.

### 수동 빈 등록 하는 상황?

애플리케이션은 크게 `업무 로직`과 `기술 지원`으로 나눌 수 있다.

- `업무 로직 빈`: 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 레포지토리 등
- `기술 지원 빈`: 기술적인 문제나 공통 관심사(AOP)를 처리

`업무 로직 빈`은 숫자도 많고 유사한 패턴이 반복되어 자동 등록 기능을 사용하는 것이 좋고,  
`기술 지원 빈`은 잘 적용 되고 있는지 아닌지 파악하기 어려운 경우가 많아 설정 정보에 나타나도록 수동으로 등록하는 것이 좋다.  
그 외에도 비즈니스 로직 중에서 다형성을 적극 활용해야하는 경우엔 수동 등록을 고민해 볼 수 있다.

###### 참고자료

- [스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술](https://www.inflearn.com/course/스프링-입문-스프링부트)
- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/스프링-핵심-원리-기본편)