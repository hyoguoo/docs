---
layout: editorial
---

# WebFlux in API Gateway

API Gateway의 주된 역할은 요청을 받아서 내부 서비스로 '전달'하고, 그 응답을 다시 클라이언트에게 '전달'하는 것이다.

## API Gateway에서 Non-Blocking 방식이 중요한 이유

API Gateway 작업 특성상 CPU 집약적인 계산보다는 대부분 I/O(네트워크 입출력) 작업에 해당한다.

- I/O Bound 작업의 특성
    - 요청을 받고 내부 서비스를 호출하면, 게이트웨이는 내부 서비스로부터 응답이 올 때까지 대기
    - 기존의 블로킹(Blocking) 방식에서는 이 '대기' 시간 동안 요청을 처리하던 스레드는 계속해서 대기
    - 많은 동시 요청이 들어올 경우, 대기하는 스레드 수가 급격히 증가하여 대규모 스레드 풀이 필요
- Non-Blocking 방식의 이점
    - WebFlux와 같은 논블로킹 모델에서는 스레드가 대기하지 않음
    - 내부 서비스에 요청을 보낸 스레드는 결과를 기다리지 않고 즉시 다른 요청을 처리하러 복귀
    - 나중에 내부 서비스로부터 응답이 오면, 이벤트 루프가 이를 감지하여 응답을 클라이언트에게 전달하는 후속 작업 처리
    - 이로 인해 소수의 스레드만으로도 수많은 동시 요청을 효율적으로 처리

## Spring Cloud Gateway에서의 WebFlux 활용 사례

Spring Cloud Gateway는 Spring 생태계에서 MSA를 위한 API Gateway 스택으로, Spring WebFlux 위에서 구축되었다.

- 기술 스택: Spring Boot, Spring WebFlux, Project Reactor 기반으로, Netty를 내장 서버로 사용하여 논블로킹 시스템으로 동작
- 동작 원리:
    1. 클라이언트의 요청이 들어오면, Netty의 이벤트 루프 스레드가 요청 수신
    2. Spring Cloud Gateway에 정의된 다양한 필터(Filter)와 라우팅 규칙(Route)들이 순차적으로 적용(인증, 로깅, 헤더 변조 등의 기능을 수행)
    3. 모든 처리는 리액티브 스트림(Mono, Flux) 파이프라인 위에서 처리
    4. 라우팅 규칙에 따라 논블로킹 HTTP 클라이언트를 사용하여 내부 마이크로서비스를 비동기적으로 호출
    5. 내부 서비스로부터 응답이 오면, 다시 리액티브 파이프라인을 통해 응답 필터들이 적용된 후 클라이언트에게 전달

## 적은 수의 스레드로 높은 처리량과 확장성을 확보하는 원리

- 스레드의 효율적 사용: 스레드가 I/O 작업을 기다리는 idle 시간이 거의 없어, 스레드를 항상 실제 작업을 처리에 사용
- 낮은 오버헤드
    - 스레드 수가 적기 때문에 스레드를 생성하고 전환하는 데 드는 컨텍스트 스위칭 비용 감소
    - 스레드 하나당 차지하는 메모리 공간(Thread Stack)도 절약되어 전체적인 시스템 리소스 사용량 감소

## 리액티브 체인으로 구현된 필터 실행

블로킹 방식의 필터와 가장 큰 차이를 보이는 핵심으로, 전통적인 필터는 단순히 다음 필터를 순차적으로 호출하지만, 리액티브 필터 체인은 `Mono`와 같은 리액티브 타입으로 연결된다.

- 각 필터는 `filter(ServerWebExchange exchange, GatewayFilterChain chain)` 메서드 구현
    - `ServerWebExchange` 매개변수: HTTP 요청과 응답에 대한 모든 정보를 포함
    - `chain` 매개변수: '다음 필터'를 포함한 나머지 체인 전체를 의미
- 'Pre' 필터 로직: `chain.filter(exchange)`를 호출하기 전의 작업
    - 다운스트림 서비스로 요청이 전달되기 전에 실행
    - 주로 요청 헤더를 추가하거나 인증을 수행하는 역할
- 'Post' 필터 로직: `chain.filter(exchange)`가 반환하는 `Mono<Void>`에 `.then()`, `flatMap` 같은 연산자를 사용하여 연결된 후속 작업
    - 다운스트림 서비스로부터 응답이 돌아온 후에 실행
    - 주로 응답 헤더를 추가하거나 응답 본문을 로깅하는 역할

```java
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    // === 'Pre' 필터 ===
    ServerHttpRequest mutatedRequest = exchange.getRequest().mutate()
            .headers(h -> h.add("X-Request-ID", "some-value"))
            .build();

    ServerWebExchange mutatedExchange = exchange.mutate()
            .request(mutatedRequest)
            .build();

    // 체인 진행(논블로킹)
    return chain.filter(mutatedExchange)
            // === 'Post' 필터 ===
            .then(Mono.fromRunnable(() -> {
                ServerHttpResponse response = mutatedExchange.getResponse();
            }));
}
```
