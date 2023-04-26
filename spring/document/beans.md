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

## 스코프

빈이 존재할 수 있는 범위를 뜻하며, 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문에 보통 스프링 컨테이너의 시작과 종료 시점과 같으며 크게 세 가지 스코프를 지원한다.

- 싱글톤(Default): 스프링 컨테이너의 시작과 종료 시점에 생성되고 소멸하는 가장 넓은 범위의 스코프
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
|    싱글톤 스코프의 빈을 스프링 컨테이너에 요청    |    프로토타입 스코프 빈을 스프링 컨테이너에 요청    |
|    (요청 한 빈이 이미 생성되어 있는 상태)     | 요청 온 시점에 프로토타입 빈을 생성하고 의존 관계 주입 |
|     스프링 컨테이너가 관리하고 있는 빈 반환     |            생성한 빈을 반환            |
| 같은 요청이 와도 계속 같은 인스턴스의 스프링 빈 반환 | 같은 요청이 오면 계속 새로운 인스턴스의 스프링 빈 반환 |

- 프로토타입 빈 특징
    - 스프링 컨테이너에 요청할 때 마다 새로운 인스턴스를 생성
    - 스프링 컨테이너는 프로토타입 스코프 빈을 `생성/의존관계 주입/초기화`까지만 처리
    - `@PreDestroy` 같은 종료 메서드도 호출하지 않음
    - 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 관리
    - 여러 빈에서 같은 프로토타입 빈을 주입 받으면, 주입 받은 모든 빈은 각자 다른 인스턴스를 사용(하나의 빈에선 계속 같은 프로토타입 빈을 가짐)

### 싱글톤 빈과 함께 사용시 항상 새로운 프로토 타입 빈을 받는 방법

싱글톤 빈에서 프로토 타입 빈을 주입 받게 되면 싱글톤 빈은 항상 같은 프로토 타입 빈을 사용하게 된다.  
만약 프로토 타입 빈을 항상 다른 빈을 사용하고 싶다면 이를 해결 하는 방법이 몇 가지 존재한다.(이 목적이라면 싱글톤 빈으로 해결하면 된다.)  
사실 실제로 프로토타입 빈을 사용하는 일은 매우 드물고, 순환참조 에러와 같은 특정 상황에서만 사용한다.

#### 항상 같은 프로토 타입 빈을 사용 중인 테스트 코드

```java
public class SingletonWithPrototypeTest {

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(SingletonBean.class, PrototypeBean.class);
        SingletonBean singletonBean1 = applicationContext.getBean(SingletonBean.class);
        int count1 = singletonBean1.logic();
        assertThat(count1).isEqualTo(1);

        SingletonBean singletonBean2 = applicationContext.getBean(SingletonBean.class);
        int count2 = singletonBean2.logic();
        assertThat(count2).isEqualTo(2);
    }

    @Scope("singleton")
    @RequiredArgsConstructor
    static class SingletonBean {
        private final PrototypeBean prototypeBean;

        public int logic() {
            prototypeBean.addCount();
            return prototypeBean.getCount();
        }
    }

    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return count;
        }
    }
}
```

#### `ObjectProvider` / `JSR-330 Provider` 사용

```java

// ObjectProvider
public class SingletonBean {

    @Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}

// JSR-330 Provider
public class SingletonBean {

    @Autowired
    private Provider<PrototypeBean> provider;

    public int logic() {
        PrototypeBean prototypeBean = provider.get();
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}
```

- 위 방법은 `ac.getBean()`을 통해서 빈을 가져오는 방법과 동일(`Dependency Lookup` 의존관계 탐색 방식)
- 기능이 단순하여 단위테스트를 만들거나 mock 코드를 만들기 쉬움
- 차이점
    - `ObjectProvider`: 스프링이 제공하는 기능으로, 외부라이브러리 필요 없이 스프링에 의존
    - `JSR-330 Provider`: 자바 표준 기능으로, `javax.inject:javax.inject:1` 라이브러리 추가 필요

### 웹 스코프

웹 스코프는 웹 환경에서만 동작하는 스코프로, 프로토타입과 다르게 스프링이 해당 스코프 종료시점까지 관리하여 종료 메서드도 호출한다.  
웹 스코프의 종류는 `request`, `session`, `application`, `websocket` 이 있다.

```java

@Component
@Scope(value = "request")
public class ExampleBean {

    // ...
}
```

웹 스코프는 위 코드처럼 생성할 수 있으며, 만약 이 `Bean`을 주입 받아 스프링 애플리케이션을 실행하게 되면 오류가 발생하게 된다.

```shell
Error creating bean with name 'exampleBean': Scope 'request' is not active for the current thread; consider defining a scoped proxy for this bean if you intend to refer to it from a singleton;
```

스프링 애플리케이션을 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, `request` 스코프는 요청이 들어오기 전의 상태기 때문에(존재하지 않음) 오류가 발생하게 된다.  
이를 위해 `Provider`을 사용해서 해결하는 방법도 있으나 `Proxy`를 사용하는 방법이 더 좋다.

#### Proxy

```java

@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ExampleBean {

    // ...
}
```

- `proxyMode`를 `TARGET_CLASS`로 설정하면 `CGLIB`를 사용해서 `Proxy`를 생성한다.
- 이렇게 하면 `ExampleBean`의 `Proxy` 객체가 생성되고, HTTP request와 상관 없이 `Proxy` 객체를 다른 빈에 미리 주입해 둘 수 있다.
- 해당 클래스를 로그로 확인해보면 `CGLIB`라는 라이브러리가 바이트코드를 조작해 `Proxy` 객체를 생성한 것을 확인할 수 있다.
- 다른 빈에서 `Example Bean`를 호출하면 실제로는 `Proxy` 객체를 호출하게 되고, 이 `Proxy` 객체가 진짜 `Example Bean`을 호출한다

```shell
Client --> Proxy Class --> Example Bean
```

- `Proxy Class`는 원본 클래스인 `Example Bean`을 상속받아서 생성되기 때문에 클라이언트는 `Example Bean`의 모든 기능을 사용할 수 있다.(다형성)

##### `Proxy Class`의 동작 방식

1. `CGLIB`가 클래스를 상속 받은 `Proxy Class`를 생성해서 주입
2. `Proxy Class`는 실제 요청이 왔을 때 원본 클래스에 요청하는 위임 로직 작동
3. `Proxy Class`는 실제 `request scope`와 관계 없으며 단순 위임 로직만 존재고 싱글톤처럼 동작

##### 특징

- `Proxy` 객체를 이용해 클라이언트는 싱글톤 빈을 사용하듯이 `request scope`을 사용할 수 있게 된다.
- 실제로 객체 조회가 꼭 필요한 시점까지 지연처리 할 수 있는 큰 특징이 있다.
- `Annotation` 설정 변경만으로 원본 객체를 `Proxy` 객체로 대체할 수 있다.

###### 참고자료

- https://www.inflearn.com/course/스프링-입문-스프링부트
- https://www.inflearn.com/course/스프링-핵심-원리-기본편