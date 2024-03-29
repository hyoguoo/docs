---
layout: editorial
---

# Beans(빈)

> 스프링 빈(Bean) : 스프링 컨테이너에 의해 생성 및 관리하는 자바 객체

Bean은 스프링을 구성하는 핵심요소로, 아래의 구성 요소를 가지고 있다.

1. class(클래스): Bean으로 등록할 Java 클래스
    - 스프링 컨테이너에 의해 객체로 생성 및 관리되어 클래스의 인스턴스를 Bean으로 사용
    - 이후 필요한 곳에서 해당 Bean을 주입하여 사용
2. id(고유 식별자): 각 Bean의 고유한 식별자
    - 스프링 컨테이너에서 각 Bean을 식별 및 조회할 때 사용
3. scope(범위): 해당 Bean이 유효 범위 정책 정의
    - Singleton / Prototype / Request / Session / Global Session 존재(기본값: Singleton)
4. constructor arguments(생성자 인자): Bean을 생성할 때 생성자에 전달할 인자들을 정의
    - Bean을 초기화하는 데 사용
    - 정의된 생성자에 대응하는 값을 넣어 Bean을 생성할 때 사용
5. property(속성): Bean을 생성할 때 setter를 통해 전달할 인자들을 정의

## 스프링 빈 등록

스프링 빈을 등록할 때 현재 주로 사용되는 방법은 크게 2가지로, 컴포넌트 스캔을 통한 등록과 `@Bean` / `@Configuration`을 통한 등록이 있다.

### 1. 컴포넌트 스캔을 통한 등록

`@Component` Annotation이 있을 경우 스프링 빈으로 자동 등록하는 방법으로 주로 사용되는 방법이다.

```java

@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}

@Service
public class MemberService {
    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}


@Repository
public class MemoryMemberRepository implements MemberRepository {
}
```

위와 같이 클래스 생성 후 클래스에 `@Component`(`@Controller` / `@Service`)을 선언하고, 생성자에 `@Autowired`를 선언하면된다.  
이렇게 하면 애플리케이션이 실행되어 객체 생성 시점에 스프링이 서비스 객체를 연결하기 위해 연관된 객체를 스프링 컨테이너에서 찾아 넣게 해준다(의존성 주입).  
의존성 주입이 완료된 후에는 아래와 같은 의존성 관계가 형성된다.

```
MemberController -> MemberService -> MemeberRepository
```

### 2. @Bean / @Configuration을 통한 등록

정형화 되지 않은 코드거나, 상황에 따라 구현 클래스를 변경해야하는 경우 사용하는 방법으로, `@Configuration` 클래스와 `@Bean`을 통해 직접 등록하는 방법이다.

```java

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

결과는 1번의 컴포넌트 스캔을 통한 등록과 동일하다.

## 빈 생명주기

데이터베이스 커넥션 풀이나 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 종료 시점에 연결을 모두 종료하는 작업을 진행하기 위해서는 객체의 초기화와 종료 작업이 필요하다.  
스프링 빈도 역시 생성 - 의존관계 주입 단계를 거쳐 사용할 수 있는 상태가 되고, 시작 및 종료 시점에 추가적인 작업을 수행할 수 있다.
스프링 빈의 대략적인 이벤트 라이프 사이클은 아래와 같다.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
    - `Bean`으로 등록할 수 있는 `Annotation` 및 `Configuration`을 읽어 IoC 컨테이너 안에 `Bean`으로 등록
    - 의존 관계 주입 전 객체 생성 과정 수행(Field/Setter 주입의 경우에만)
        - 생성자 주입: 객체의 생성과 의존관계 주입이 동시에 발생
        - Setter/Field 주입: 객체 생성 --> 의존 관계 주입 단계 분리
3. [의존 관계 주입](dependency-injection)
4. 초기화 콜백
    - 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
5. 사용
    - 실제 애플리케이션(빈) 동작 단계
6. 소멸 전 콜백
    - 빈이 소멸되기 직전에 호출
7. 스프링 종료

### 빈 생명주기(초기화/소멸) 콜백

시작 및 종료 시점에 추가적인 작업을 수행하기 위해 스프링은 다양한 방법을 지원한다.  
스프링에서는 크게 3가지 방법으로 빈 생명주기 콜백을 지원하며 `@PostConstruct`, `@PreDestroy`를 가장 권장하고 있다.  
실행 되는 순서는 아래 번호 순서대로 실행된다.

#### 1. @PostConstruct, @PreDestroy 애노테이션

- @PostConstruct: 의존관계 주입이 끝나면 호출
- @PreDestroy: 빈이 종료될 때 호출

```java
public class Example {
    // ...

    @PostConstruct
    public void init() {
        action();
    }

    @PreDestroy
    public void close() {
        exit();
    }
}
```

- 최신 스프링에서 가장 권장하는 방법
- `javax.annotation.PostConstruct`인 자바 표준 기술
- 컴포넌트 스캔과 궁합이 좋음
- 외부 라이브러리에는 적용 불가능 -> 이 경우 `@Bean`의 `initMethod`, `destroyMethod`를 사용

#### 2. 인터페이스(InitializingBean, DisposableBean)

- `InitializingBean` In `afterPropertiesSet()`: 의존관계 주입이 끝나면 호출
- `DisposableBean` In `destroy()`: 빈이 종료될 때 호출

```java
public class Example implements InitializingBean, DisposableBean {
    // ...

    @Override
    public void afterPropertiesSet() throws Exception {
        action();
    }

    @Override
    public void destroy() throws Exception {
        exit();
    }
}
```

- 스프링 전용 인터페이스에 의존
- 초기화, 소멸 메서드의 이름 변경 불가
- 코드를 고칠 수 없는 외부 라이브러리에 적용 불가

#### 3. 설정 정보에 초기화 메서드, 종료 메서드 지정

- `@Bean(initMethod = "init", destroyMethod = "close")` 초기화/소멸 메서드 지정

```java
public class Example {
    // ...

    public void init() {
        action();
    }

    public void close() {
        exit();
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public Example example() {
        return new Example();
    }
}
```

- 메서드 이름 자유롭게 설정 가능
- 스프링 빈이 스프링 코드에 의존하지 않음
- 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드 적용 가능
- 종료 메서드 추론 기능
    - `@Bean`의 `destroyMethod` 속성에는 아무것도 지정하지 않으면 추론 기능이 동작
    - `close`, `shutdown` 같은 일반적으로 많이 사용하는 종료 메서드를 자동으로 호출
    - 추론 기능을 막기 위해서 `destroyMethod=""`로 지정 할 수 있음

## 스코프

빈이 존재할 수 있는 범위를 뜻하며, 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문에 보통 스프링 컨테이너의 시작과 종료 시점과 같으며 크게 세 가지 스코프를 지원한다.

- 싱글톤(Default): 스프링 컨테이너의 시작과 종료 시점에 생성되고 소멸하는 가장 넓은 범위의 스코프, 가장 많이 사용되는 스코프
- 프로토타입: 스프링 컨테이너가 빈의 생성과 의존 관계 주입까지만 관여하고 더는 관리하지 않는 스코프
- 웹 관련 스코프
    - request: 웹 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프
    - session: 웹 Session과 동일한 생명주기를 가지는 스코프
    - application: 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프

### 프로토타입 스코프

싱글톤 스코프는 스프링 컨테이너에서 항상 같은 인스턴스의 스프링 빈을 반환하지만, 스코프의 빈은 스프링 컨테이너에 요청할 때마다 새로 인스턴스를 생성하여 반환한다.

- 싱글톤 빈 / 프로토타입 빈 비교

|             싱글톤 빈              |             프로토타입 빈             |
|:------------------------------:|:-------------------------------:|
|    싱글톤 스코프 빈을 스프링 컨테이너에 요청     |    프로토타입 스코프 빈을 스프링 컨테이너에 요청    |
|    (요청 한 빈이 이미 생성되어 있는 상태)     | 요청 온 시점에 프로토타입 빈을 생성하고 의존 관계 주입 |
|     스프링 컨테이너가 관리하고 있는 빈 반환     |            생성한 빈을 반환            |
| 같은 요청이 와도 계속 같은 인스턴스의 스프링 빈 반환 | 같은 요청이 오면 계속 새로운 인스턴스의 스프링 빈 반환 |

- 프로토타입 빈 특징
    - 스프링 컨테이너에 요청할 때 마다 새로운 인스턴스를 생성
    - 스프링 컨테이너는 프로토타입 스코프 빈을 `생성/의존관계 주입/초기화`까지만 처리
    - 생성 및 초기화까지만 처리하기 때문에 `@PreDestroy` 같은 종료 메서드도 호출하지 않음
    - 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 관리
    - 여러 빈에서 같은 프로토타입 빈을 주입 받으면, 주입 받은 모든 빈은 각자 다른 인스턴스를 사용(하나의 빈에선 계속 같은 프로토타입 빈을 가짐)

[//]: # (### 웹 스코프)

[//]: # ()
[//]: # (웹 스코프는 웹 환경에서만 동작하는 스코프로, 프로토타입과 다르게 스프링이 해당 스코프 종료시점까지 관리하여 종료 메서드도 호출한다.  )

[//]: # (웹 스코프의 종류는 `request`, `session`, `application`, `websocket` 이 있다.)

[//]: # ()
[//]: # (```java)

[//]: # ()
[//]: # (@Component)

[//]: # (@Scope&#40;value = "request"&#41;)

[//]: # (public class ExampleBean {)

[//]: # ()
[//]: # (    // ...)

[//]: # (})

[//]: # (```)

[//]: # ()
[//]: # (웹 스코프는 위 코드처럼 생성할 수 있으며, 만약 이 `Bean`을 다른 곳에 주입 하여 스프링 애플리케이션을 실행하게 되면 오류가 발생하게 된다.)

[//]: # ()
[//]: # (```shell)

[//]: # (Error creating bean with name 'exampleBean': Scope 'request' is not active for the current thread; consider defining a scoped proxy for this bean if you intend to refer to it from a singleton;)

[//]: # (```)

[//]: # ()
[//]: # (스프링 애플리케이션을 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, `request` 스코프는 실제 요청이 들어오기 전의 상태기 때문에&#40;존재하지 않음&#41; 오류가 발생하게 된다.  )

[//]: # (이를 위해 `Provider`을 사용해서 해결하는 방법도 있으나 `proxyMode`를 사용하는 방법이 더 간단하다.)

[//]: # ()
[//]: # (#### proxyMode)

[//]: # ()
[//]: # (```java)

[//]: # ()
[//]: # (@Component)

[//]: # (@Scope&#40;value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS&#41; // proxyMode 추가)

[//]: # (public class ExampleBean {)

[//]: # ()
[//]: # (    // ...)

[//]: # (})

[//]: # (```)

[//]: # ()
[//]: # (- `proxyMode`를 `TARGET_CLASS`로 설정하면 `CGLIB`에서 사용해서 프록시 객체를 생성한다.)

[//]: # (- 생성된 프록시 객체는 싱글톤처럼 동작하여 HTTP request와 상관 없이 다른 빈에 미리 주입해 둘 수 있다.)

[//]: # (- 다른 빈에서 `ExampleBean`를 호출하면 실제로는 프록시 객체를 호출하게 되고, 이 프록시 객체가 `ExampleBean`을 호출하게 된다.)

[//]: # (- 프록시 클래스는 원본 클래스인 `ExampleBean`을 상속받아서 생성되기 때문에 클라이언트는 `ExampleBean`의 모든 기능을 사용할 수 있다.&#40;다형성&#41;)

###### 참고자료

- [스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술](https://www.inflearn.com/course/스프링-입문-스프링부트)
- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/스프링-핵심-원리-기본편)
