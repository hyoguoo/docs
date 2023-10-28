---
layout: editorial
---

# Validation(검증)

> Bean Validation은 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380) 기술 표준

검증 로직을 모든 프로젝트에 적용할 수 있게 공통화 및 표준화 한 것으로, 이를 잘 활용하면, 애노테이션 하나(`@Valid` / `@Validated`)로 검증 로직을 매우 편리하게 적용할 수 있다.

```java
public class Example {
    @NotNull
    @Min(0)
    Integer example;
}
```

```java
public class Example {
    @PostMapping("/example")
    public String example(@Valid @ModelAttribute Example example, BindingResult bindingResult) {
        // ...

        return "example";
    }
}
```

## @Valid

JSR-303(자바) 표준 스펙으로, 빈 검증기(`Bean Validator`)를 이용해 제약 조건을 검증한다.

### 동작 원리

서버로 오는 요청은 아래의 프로세스를 거치게 된다.

1. 프론트 컨트롤러(디스패처 서블릿)을 통해 컨트롤러로 전달
2. 컨트롤러 메서드의 객체를 만들어주는 Argument Resolver 동작
3. ArgumentResolver에서 @Valid 어노테이션이 있는 경우 해당 객체에 대한 검증을 수행
4. 검증에 실패하면 `MethodArgumentNotValidException` 예외 발생

Argument Resolver에서 검증을 수행하기 때문에 컨트롤러에서만 동작하게 되며, 컨트롤러가 아닌 곳에서는 동작하지 않는다.

## @Validated

Spring 프레임워크에서 제공하는 애노테이션으로, 컨트롤러가 아닌 곳에서도 검증을 수행할 수 있다.  
컨트롤러가 아닌 곳에서 검증을 수행하려면 `@Validated`를 클래스 레벨에 선언한 뒤, 검증을 수행하고자 하는 메서드에 파라미터에 `@Valid`를 선언해야 한다.

```java

@Service
@Validated
public class ExampleService {

    public void example(@Valid Example example) {
        // ...
    }
}
```

### 동작 원리(클래스 레벨에 선언한 경우)

Validated는 AOP 기반으로 메서드 요청을 인터셉터하여 처리하게 된다.

1. 해당 클래스에 유효성 검증을 위한 인터셉터(MethodValidationInterceptor) 등록
2. 해당 클래스의 메서드들이 호출될 때 요청을 가로채서 유효성 검증 수행
3. 검증에 실패하면 `ConstraintViolationException` 예외 발생

## @Valid vs @Validated

|                 @Valid                  |              @Validated              |
|:---------------------------------------:|:------------------------------------:|
|              JSR-303 표준 스펙              |          Spring 프레임워크에서 제공           |
|        ArgumentResolver에서 검증을 수행        |           AOP 기반으로 검증을 수행            |
|               컨트롤러에서만 동작                |          컨트롤러가 아닌 모든 곳에서 동작          |
| `MethodArgumentNotValidException` 예외 발생 | `ConstraintViolationException` 예외 발생 |

###### 참고자료

- [망나니개발자 티스토리](https://mangkyu.tistory.com/174)