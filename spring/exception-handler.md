---
layout: editorial
---

# Exception Handler(예외 핸들러)

Spring MVC는 컨트롤러에서 발생한 예외를 일관된 규칙으로 HTTP 응답으로 변환하는데, 동작의 기본 축은 `HandlerExceptionResolver` 체인이다.

## 동작 흐름

1. DispatcherServlet 감지
2. HandlerExceptionResolver 체인 처리
3. 로컬 @ExceptionHandler
4. 전역 @ControllerAdvice
5. 기본 Resolver
6. 미해결 시 /error

예외는 DispatcherServlet에서 감지되고, 등록된 HandlerExceptionResolver 체인으로 전달되면서 다음과 같은 우선 순위로 처리된다.

1. `ExceptionHandlerExceptionResolver`: @ExceptionHandler 메서드를 탐색해 호출
2. `ResponseStatusExceptionResolver`: @ResponseStatus 애노테이션 또는 ResponseStatusException 처리
3. `DefaultHandlerExceptionResolver`: 표준 Spring MVC 예외를 적절한 상태 코드로 변환

## @ExceptionHandler

컨트롤러 내부에 @ExceptionHandler 메서드를 선언해 특정 예외를 처리할 수 있다.

```java

@RestController
@RequestMapping("/orders")
public class OrderController {

    @GetMapping("/{id}")
    public Order find(@PathVariable Long id) {
        if (id < 0)
            throw new IllegalArgumentException("invalid id");
        throw new OrderNotFoundException(id);
    }

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<Map<String, Object>> notFound(OrderNotFoundException e, HttpServletRequest req) {
        Map<String, Object> body = new LinkedHashMap<>();
        body.put("code", "ORDER_NOT_FOUND");
        body.put("message", e.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> badRequest(IllegalArgumentException e) {
        return ResponseEntity.badRequest().body(e.getMessage());
    }
}
```

- @ExceptionHandler 메서드는 컨트롤러 내부에 위치하며, 지정한 예외 타입을 처리
- 메서드 매개변수로 예외 객체, HttpServletRequest, HttpServletResponse, Model 등을 받을 수 있음
- 반환 타입은 ResponseEntity, ModelAndView, 단순 객체 등 다양하게 지정 가능
- 상속 관계에 있는 예외는 가장 구체적인 타입의 핸들러가 우선 호출

## @ControllerAdvice / @RestControllerAdvice

@ControllerAdvice로 전역 규칙을 구성하면 관리 비용을 줄일 수 있다.(@RestControllerAdvice는 @ResponseBody를 포함해 응답을 본문으로 직렬화)

```java

@Order(Ordered.HIGHEST_PRECEDENCE)
@RestControllerAdvice(basePackages = "com.example.api")
public class ApiExceptionAdvice {

    @ExceptionHandler(RacingCarException.class)
    public ResponseEntity<ErrorResult> handle(RacingCarException e) {
        return ErrorResult.toResponseEntity(String.valueOf(e.getStatus().value()), e.getMessage());
    }
}
```

- ExceptionHandlerExceptionResolver가 애플리케이션 컨텍스트의 @ControllerAdvice 빈을 스캔
- 스코프(basePackages, basePackageClasses, annotations)가 일치하는 컨트롤러 요청에 적용
- 동일 예외에 대해 로컬 @ExceptionHandler(컨트롤러 내부)가 존재하면 로컬 핸들러가 우선
- 여러 Advice가 동시에 매칭되면 @Order/Ordered 우선순위에 따라 선택
