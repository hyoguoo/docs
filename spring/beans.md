# 스프링 빈

> 스프링 빈(Bean) : 스프링 컨테이너가 관리하는 자바 객체

`new`를 통해 생성하는 것이 아닌 컨테이너에서 스스로 생성하고 관리하는 객체

## 스프링 빈 등록

### 1. [컴포넌트 스캔](container.md)과 자동 의존 관계를 통한 등록

@Component Annotation이 있을 경우 스프링 빈으로 자동 등록

```java

@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

위와 같이 Controller Package에 클래스 생성 후 `@Controller` Annotation을 넣고,  
서비스 객체를 연결하기 위해 `@Autowired` Annotation을 넣어 스프링이 연관된 객체를 스프링 컨테이너에서 찾아 넣게 해준다(의존성 주입).

그 후 서비스 객체와 레포지토리를 연결하기 위해(컨테이너에 등록하기 위해) `@Service` / `@Repository` Annotation을 통해 스프링 빈으로 등록해준다.

```java

@Service
public class MemberService {
    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

```java

@Repository
public class MemoryMemberRepository implements MemberRepository {
}
```

모두 등록하게 되면 스프링 컨테이너는 아래의 형태를 띄게 된다.

```
MemberController -> MemberService -> MemeberRepository
```

** 생성자에 `@Autowired`를 사용하면 객체 생성 시점에 스프링 컨테이너에서 해당 스프링 빈을 찾아서 주입하며, 생성자가 1개만 있을 경우엔 `@Autowired`를 생략할 수 있다.  
** 스프링 컨테이너에 스프링 빈을 등록할 때 기존적으로 싱글톤으로 등록하며, 다른 설정을 통해 다른 패턴으로 생성할 수 있지만 대부분 싱글톤을 사용

### 2. 자바 코드를 통한 직접 등록

정형화 되지 않은 코드거나, 상황에 따라 구현 클래스를 변경해야하는 경우 사용  
Config Class를 생성해 `@Configuration` Annotation을 붙여 등록할 객체들을 `@Bean` Annotation을 통해 직접 등록

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

## 빈 생명주기

데이터베이스 커넥션 풀이나 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 종료 시점에 연결을 모두 종료하는 작업을 진행하기 위해서는 객체의 초기화와 종료 작업이 필요하다.   
스프링 빈도 위와 비슷하게 생성과 의존관계 주입이라는 라이프 사이클을 가지게 되며 그 이후에 필요한 데이터를 사용할 수 있는 준비가 완료된 상태가 된다.

### 스프링 빈의 이벤트 라이프 사이클

1. 스프링 컨테이너 생성
    - IoC 컨테이너 생성
2. 스프링 빈 생성
    - `Bean`으로 등록할 수 있는 `Annotation` 및 `Configuration`을 읽어 IoC 컨테이너 안에 `Bean`으로 등록
    - 의존 관계 주입 전 객체 생성 과정 수행(Field/Setter 주입의 경우에만)
        - 생성자 주입: 객체의 생성과 의존관계 주입이 동시에 발생
        - Setter/Field 주입: 객체 생성 --> 의존 관계 주입 단계 분리
3. [의존 관계 주입](dependency_injection.md)
4. 초기화 콜백
    - 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
5. 사용
    - 실제 애플리케이션(빈) 동작 단계
6. 소멸전 콜백
    - 빈이 소멸되기 직전에 호출
7. 스프링 종료

### 빈 생명주기(초기화/소멸) 콜백

스프링에서는 크게 3가지 방법으로 빈 생명주기 콜백을 지원하며 현재는 마지막 방법을 권장한다.

#### 1. 인터페이스(InitializingBean, DisposableBean)

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

#### 2. 설정 정보에 초기화 메서드, 종료 메서드 지정

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
- 종료 메서드 추론
    - `@Bean`의 `destroyMethod` 속성에는 아무것도 지정하지 않으면 추론 기능이 동작(`destroyMethod = "(inferred)"`)
    - `close`, `shutdown` 메서드를 자동으로 호출
    - 추론 기능을 사용하지 않으려면 `destroyMethod=""`로 지정

#### 3. @PostConstruct, @PreDestroy 애노테이션 지원

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

### 객체 생성

###### 출처

- https://www.inflearn.com/course/스프링-입문-스프링부트