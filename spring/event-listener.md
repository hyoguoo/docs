---
layout: editorial
---

# EventListener

## 스프링 이벤트

스프링 이벤트는 옵저버(Observer) 패턴의 구현체로, 애플리케이션 내에서 발생하는 특정 상태 변화나 이벤트를 다른 객체에 알리기 위해 사용된다.

- `ApplicationEvent`: 발생하는 모든 이벤트의 부모 클래스(커스텀 이벤트를 정의할 때 상속)
- `ApplicationEventPublisher`: 이벤트를 발행하는 역할을 하는 인터페이스
- `@EventListener`: 이벤트를 수신하여 처리할 메서드에 적용하는 어노테이션

이를 통해 이벤트 발행자(Publisher)와 이벤트 구독자(Listener) 간의 결합도를 낮출 수 있다.

## 동작 원리

스프링의 이벤트 메커니즘은 `ApplicationContext`가 `ApplicationEventPublisher` 인터페이스를 구현하는 것에서 시작된다.

1. 리스너 등록: 스프링 컨테이너가 초기화될 때, `BeanPostProcessor`가 빈으로 등록된 객체들의 모든 메서드를 스캔하여 `@EventListener` 어노테이션이 붙은 메서드 조회
2. 리스너 정보 저장: 찾아낸 리스너 메서드와 해당 메서드가 구독할 이벤트 타입을 매핑하여 `ApplicationListener` 형태로 래핑한 뒤, 컨테이너 내부에 저장
3. 이벤트 발행: `publishEvent()` 메서드가 호출 시, 컨테이너는 발행된 이벤트 타입과 그 하위 타입의 이벤트를 구독하기로 등록된 모든 `ApplicationListener`를 조회
4. 리스너 실행: 찾아낸 리스너들의 메서드를 순차적 호출

### 이벤트 구독 방식

- 기본적으로 이벤트 구독은 리스너 메서드의 파라미터 타입을 기준으로 결정
- 이벤트 타입의 상속 관계가 포함되어, 발행된 이벤트가 메서드 파라미터의 타입과 일치하거나 하위 타입일 경우에도 해당 메서드가 호출
- 추가로 `@EventListener`의 `condition` 속성에 SpEL 표현식을 지정하여, 이벤트 필드 값을 기반으로 세밀한 필터링이 가능

### 리스너 순서 지정

동일 이벤트를 여러 리스너가 구독할 경우, 실행 순서는 기본적으로 정의되지 않으며, `@Order` 애너테이션을 통해 우선순위를 지정할 수 있다.

```java

@Component
public class SampleEventListener {

    @EventListener
    @Order(0) // 가장 먼저 실행
    public void handleFirst(MyEvent event) {
        // 선처리 로직
    }

    @EventListener
    @Order(100) // 나중에 실행
    public void handleSecond(MyEvent event) {
        // 후처리 로직
    }
}
```

### 비동기 처리

- 기본 이벤트 처리는 동기 방식으로, 이벤트 리스너 메서드는 이벤트를 발행한 호출 스레드 내에서 실행
- `@Async` 어노테이션을 리스너 메서드에 적용하여 별도의 스레드 풀에서 비동기적으로 실행 가능
- 비동기 처리를 통해 호출자의 응답 속도 지연을 방지하고, 이메일 발송·외부 API 연동 등 시간이 오래 걸리는 작업 분리 가능
- 비동기 실행 시 발생한 예외는 호출자에게 전파되지 않으므로, 로깅 및 별도의 예외 처리 전략 필요

### 예외 처리

리스너에서 예외가 발생하면, 동기 처리와 비동기 처리에 따라 다르게 동작한다.

|   구분    | 동작                                         | 주의 사항                             |
|:-------:|:-------------------------------------------|:----------------------------------|
| 동기 리스너  | 트랜잭션 내에서 실행, 예외 발생 시 트랜잭션 전체 롤백            | 무거운 외부 호출이나 불안정한 작업은 피하는 것이 바람직함  |
| 비동기 리스너 | `@Async` 적용 시 별도 스레드에서 실행, 호출자 트랜잭션에 영향 없음 | 예외가 호출자에게 전파되지 않아 별도의 예외 처리 전략 필요 |

## @TransactionalEventListener

```java

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@EventListener
public @interface TransactionalEventListener {

    /**
     * Phase to bind the handling of an event to.
     * <p>The default phase is {@link TransactionPhase#AFTER_COMMIT}.
     * <p>If no transaction is in progress, the event is not processed at
     * all unless {@link #fallbackExecution} has been enabled explicitly.
     */
    TransactionPhase phase() default TransactionPhase.AFTER_COMMIT;

    /**
     * Whether the event should be handled if no transaction is running.
     */
    boolean fallbackExecution() default false;

    // ... other attributes ...
}
```

일반 `@EventListener`는 트랜잭션의 성공 여부와 관계없이 이벤트를 즉시 발행하지만,`@TransactionalEventListener`는 트랜잭션의 특정 시점에 맞추어 이벤트를 처리할 수 있다.

|      Phase       | 설명                                        | 주의 사항                                                  |
|:----------------:|:------------------------------------------|:-------------------------------------------------------|
|   AFTER_COMMIT   | 트랜잭션이 성공적으로 커밋된 후에 리스너 실행 (기본값, 가장 많이 사용) | 롤백 시 실행되지 않음, 커밋 이후 보장된 상태에서 처리할 작업(외부 API, 이메일 등)에 적합 |
|  AFTER_ROLLBACK  | 트랜잭션이 롤백된 후에 리스너 실행                       | 보상 트랜잭션 로직에 활용 가능                                      |
| AFTER_COMPLETION | 트랜잭션의 커밋/롤백 여부와 상관없이 완료된 후에 리스너 실행        | 커밋/롤백 여부와 상관없이 실행되므로 후처리에 적합                           |
|  BEFORE_COMMIT   | 트랜잭션이 커밋되기 직전에 리스너 실행                     | 예외 발생 시 전체 롤백, 가벼운 검증 로직이나 트랜잭션 내에서 처리할 작업에 적합         |

기본적으로 현재 진행 중인 트랜잭션이 없는 경우 이벤트를 무시하니 주의해야 한다.(없는 상황에서도 리스너를 실행해야 한다면 `fallbackExecution = true` 설정)

## 예시 코드

사용자가 가입했을 때 이메일을 보내는 예시 코드는 다음과 같은 구조로 작성할 수 있다.

### 1. 이벤트 정의

```java
public record UserRegistrationEvent(
        String email,
        String username
) {

}
```

### 2. 이벤트 발행

```java

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public void registerUser(String email, String username) {
        // 사용자 저장 로직...
        User savedUser = userRepository.save(new User(email, username));

        // 이벤트 발행
        eventPublisher.publishEvent(new UserRegistrationEvent(savedUser.getEmail(), savedUser.getUsername()));
    }
}
```

### 3. 이벤트 리스너

```java
// UserEventListener.java
@Component
@RequiredArgsConstructor
public class UserEventListener {

    private final EmailService emailService;

    // 트랜잭션이 성공적으로 커밋된 후에 비동기로 실행
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    @Async
    public void handleUserRegistrationEvent(UserRegistrationEvent event) {
        // 이메일 발송 로직
        emailService.sendWelcomeEmail(event.email(), event.username());
    }
}
```
