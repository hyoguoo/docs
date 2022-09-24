# 의존성 주입(DI)
객체간 의존성을 자신이 아닌 외부에서 두 객체 간의 관계를 결정해주는 디자인 패턴으로.  
올바른 방법으로 의존성을 주입했을 경우 아래의 장점을 얻을 수 있으며 아래와 같은 장점이 있다.
- Test 용이
- 코드 재사용성 증가
- 객체 간 의존성 감소
- 객체 간 결합도를 낮춰 유연한 코드 작성

## DI 3가지 방법(Spring)
### 1. 생성자 주입
```java
@Controller
public class MemberController {
    private MemberService memberService;

    @Autowired // 생략가능
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```
만약 생성자가 하나라면 `@Autowired` 생략 할 수 있다.

### 2. Field 주입
```java
@Controller
public class MemberController {
    @Autowired
    private MemberService memberService;
}
```
필드에 `@Autowired` Annotation을 붙여 주입하는 방법으로 제일 간단하지만 아래의 단점이 존재한다.
- 외부에서 변경이 힘듬
- 프레임워크에 의존적이며 객체지향적으로 좋지 않음

### 3. 수정자 주입(Setter Injection)
```java
@Controller
public class MemberController {
    private MemberService memberService;

    @Autowired // 생략가능
    public void setMemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```
Setter 메서드에 `@Autowired` Annotation을 붙이는 방법으로 거의(아예) 사용하지 않는다.  
setter를 public으로 열어두어야 하리 때문에 실행 중에 의존 관계가 변경 될 가능성이 있다.

## 의존성 주입 방법 중 권장하는 방법
기본적으로 `Spring Framework`에서 권장하는 방법은 1번의 `생성자 주입`이다.  
3번의 Setter방식은 실행 중 의존성이 변경되는 위험한 가능성이 있고, 2번의 Field 주입은 아주 간단하지만 생성자 주입과는 아래의 차별점이 있다.
### 1. 순환 참조 방지
개발 단위가 커지다 보면 여러 `Component` 간에 의존성이 생기며 A <-> B 컴포넌트가 아래와 같이 서로 참조하는 코드를 작성하게 되는 경우도 생기게 된다.
```java
@Service
public class AService {
    @Autowired
    private BService bService;

    public void HelloWorldA() {
        bService.HelloWorldB();
    }
}
```
```java
@Service
public class BService {
    @Autowired
    private AService bService;

    public void HelloWorldB() {
        bService.HelloWorldA();
    }
}
```
위와 같이 됐을 때, 계속해서 객체를 서로 참조하는 메서드가 실행되어 무한루프가 되어버리기 때문에 Stack에 계속 쌓이다가 메모리가 터져버려 스택 오버플로우 에러가 발생한다.
```shell
java.lang.StackOverflowError: null
```
생성자 주입의 경우 애플리케이션을 실행하는 시점에서 발생을 시켜 해당 오류를 방지할 수 있지만, 필드 주입 및 수정자 주입을 했을 경우엔 해당 코드가 직접 호출되기 전까지는 에러를 발생시키지 않는 차이점이 있다.

### 2. 불변성
생성자로 의존성을 주입할 경우 `final`로 선언할 수 있어, 런타임 도중에 의존성을 주입 받는 객체가 바뀔 일이 없어진다.(OOP 5가지 원칙 중, 개방-폐쇄 원칙)
```java
@Controller
public class MemberController {
    private final MemberService memberService; // 불변성 보장

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

### 3. 테스트 용이
테스트가 특정 프레임워크(Spring)에 의존하는 것은 침투적이므로 좋지 않고, 순수 자바 코드로 테스트를 작성하는 것이 가장 좋다.  
그 부분에서 생성자 주입이 아닌 다른 주입 방법을 사용했을 경우 순수 자바 코드로 Unit 테스트를 작성하는 것이 어려워진다.
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
예를 들어 위와 같이 필드를 통한 의존성 주입을 했을 경우, Spring 위에서 동작하지 않는 테스트 코드에선 의존 관계 주입이 되지 않아 테스트 코드의 `memberService는` `null`이 되어 정상적인 테스트가 불가능해진다.  
만약 Setter 주입을 사용하면 불변성이 꺠지게 되며, 테스트 코드를 Spring 프레임 워크 위에서 돌리게 되면 테스트 비용(시간)이 증가하게 된다.  