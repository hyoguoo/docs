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

###### 출처

- https://www.inflearn.com/course/스프링-입문-스프링부트