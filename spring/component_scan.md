---
layout: editorial
---

# Component Scan(컴포넌트 스캔)

스프링은 스프링 빈을 손쉽게 등록하기 위해 컴포넌트 스캔이라는 기능을 제공하는데, 컴포넌트 스캔은 클래스패스를 검색하여 `@Component` 어노테이션이 붙은 클래스를 찾아 빈으로 등록하게 된다.

```java
// AppConfig.java
// @ComponentScan 사용하여 @Component 어노테이션이 붙은 클래스를 스캔하여 빈으로 등록  
@Configuration
@ComponentScan(
        // 스캔할 패키지의 시작 위치를 지정, 이 패키지를 포함해서 하위 패키지를 모두 탐색
        basePackages = {"hello.core", "hello.service"},
        // 지정한 클래스의 패키지를 탐색 시작 위치로 지정, 지정하지 않으면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치로 설정됨
        basePackageClasses = AutoAppConfig.class
)
public class AutoAppConfig {
}

// OrderServiceImpl.java
// @Component 어노테이션을 선언하여 @ComponentScan의 대상이 되도록 설정
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy
            discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

위 처럼 패키지 위치를 지정할 수 있지만 일반적으로 설정 정보(`AppConfig`) 클래스의 위치를 프로젝트 최상단에 두는 것을 권장한다.
아래는 컴포넌트 스캔의 대상 `Annotation`들로, 내부 구현 코드를 보면 모두 `@Component`을 포함하고있다.

- `@Component` : 컴포넌트 스캔의 대상에 사용
- `@Controller` : 스프링 MVC 컨트롤러에서 사용 -> 스프링 MVC 컨트롤러로 인식
- `@Service` : 스프링 비즈니스 로직에서 사용 -> 특별한 처리는 없으나 의미상 구분하기 위해 사용
- `@Repository` : 스프링 데이터 접근 계층에서 사용 -> 데이터 계층(Database)의 예외를 스프링 예외로(추상화) 변환
- `@Configuration` : 스프링 설정 정보에서 사용 -> 스프링 설정 정보로 인식하며, 스프링 빈이 싱글톤을 유지하도록 추가 처리

## 필터

컴포넌트 스캔 대상을 추가로 지정하거나, 제외할 대상을 지정할 수 있는데 `@Component`의 사용으로 충분하기 때문에 필터를 사용할 일은 거의 없다.  
`excludeFilters`를 사용하는 경우가 있긴 하지만, 권장하지 않으며 스프링 기본 설정에 맞추어 사용하는 것이 좋다.

```java
// IncludeComponent.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}

// ExcludeComponent.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyIncludeComponent {
}

// AppConfig.java
@Configuration
@ComponentScan(
        // 컴포넌트 스캔 대상에 추가할 애노테이션
        includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
        // 컴포넌트 스캔 대상에서 제외할 애노테이션
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
)
public class AutoAppConfig {
}
```

FilterType에 줄 수 있는 옵션은 아래와 같다.

- `ANNOTATION`: 기본값, 애노테이션을 인식해서 동작
- `ASSIGNABLE_TYPE`: 지정한 타입과 자식 타입을 인식해서 동작
- `ASPECTJ`: AspectJ 패턴 사용
- `REGEX`: 정규 표현식
- `CUSTOM`: `TypeFilter`이라는 인터페이스를 구현해서 처리

## 의존관계 주입 동작 방식

### 1. `@ComponentScan`

`@ComponentScan`은 `@Component`가 붙은 클래스를 스캔하여 빈으로 등록한다.

- java 코드

```java

@Component
public class MemberServiceImpl implements MemberService {
}

@Component
public class OrderServiceImpl implements OrderService {
}

@Component
public class MemoryMemberRepository implements MemberRepository {
}

@Component
public class RateDiscountPolicy implements DisCountPolicy {
}
```

- 스프링 빈 저장소

|          빈 이름          |            빈 객체            |
|:----------------------:|:--------------------------:|
|   memberServiceImpl    |   MemberServiceImpl@x01    |
|    orderServiceImpl    |    OrderServiceImpl@x02    |
| memoryMemberRepository | MemoryMemberRepository@x03 |
|   rateDiscountPolicy   |   RateDiscountPolicy@x04   |

스프링 빈의 기본 이름은 클래스명을 사용하되, 맨 앞글자만 소문자로 바꾼 이름을 사용한다.  
(만약 직접 지정하고 싶다면 `@Component("memberService2")`와 같이 지정할 수 있다.)

### 2. `@Autowired`

생성자에 `@Autowired`를 지정하면, 스프링 컨테이너가 스프링 빈(타입이 같은 빈)을 찾아서 해당 파라미터에 넣어준다.  
생성자가 하나인 경우 생략할 수 있기 때문에 그 경우엔 보통은 생략한다.

```java

@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired // 생성자가 하나일 경우 생략 가능하기 때문에 보통은 생략하는 경우가 많다.
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy
            discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

###### 참고자료

- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/스프링-핵심-원리-기본편)