---
layout: editorial
---

# Spring Boot Test Context(스프링 부트 테스트 컨텍스트)

스프링 부트는 다양한 계층의 테스트를 지원하기 위해 여러 가지 테스트 애노테이션을 제공하여, 테스트 환경을 쉽게 구성할 수 있도록 도와준다.

| 애노테이션             | 주요 특징                                                     | 사용 상황                                                            |
|-------------------|-----------------------------------------------------------|------------------------------------------------------------------|
| `@SpringBootTest` | - 전체 애플리케이션 컨텍스트 로드<br/>- 통합 테스트 수행<br/>- 모든 계층 테스트 가능    | - 애플리케이션 전체 흐름 테스트<br/>- 여러 계층 간 상호작용 테스트<br/>- 실제와 유사한 환경에서 테스트 |
| `@DataJpaTest`    | - JPA 관련 Bean만 로드<br/>- 트랜잭션 관리 및 롤백<br/>- 내장형 데이터베이스 사용  | - JPA 엔티티 및 리포지토리 테스트<br/>- 데이터베이스 쿼리 검증<br/>- JPA 관련 테스트 수행 시   |
| `@JdbcTest`       | - JDBC 관련 Bean만 로드<br/>- 트랜잭션 관리 및 롤백<br/>- 내장형 데이터베이스 사용 | - 순수 JDBC 테스트<br/>- SQL 쿼리 검증<br/>- JPA를 사용하지 않는 데이터베이스 상호작용 테스트 |
| `@WebMvcTest`     | - 웹 계층 관련 Bean만 로드<br/>- 내장형 웹 서버(MockMvc) 사용             | - 컨트롤러 계층 테스트<br/>- 웹 계층의 단위 테스트 수행<br/>- 빠른 웹 계층 테스트 필요 시       |

특징에서 알 수 있듯이 컨텍스트를 로드해 편리하게 테스트 환경을 제공해주며, 테스트 단위 및 목적에 따라 적절한 애노테이션을 사용하면 된다.

## 테스트 애노테이션 사용 시 주의사항

하지만 편리한만큼 이를 남용하게 되면 아래의 이유로 테스트 실행 속도가 느려지는 문제가 발생할 수 있다.

- 불필요한 로드: 테스트 목적에 맞지 않는 불필요한 Bean과 컨텍스트가 로드
- 반복 되는 로드: 같은 테스트 어노테이션이더라도 반복적으로 애플리케이션 컨텍스트를 생성함

이러한 문제를 해결하기 위한 최적화 방법은 다음과 같다.

### 애플리케이션 컨텍스트가 필요한 경우만 로드

불필요하게 컨텍스트를 로드하는 것은 테스트 실행 속도를 느리게하는 가장 큰 요인이므로, 아래와 같은 방법으로 최소한의 컨텍스트만 로드하도록 한다.

- `@SpringBootTest` 대신 `@DataJpaTest`, `@WebMvcTest` 등을 사용
- Junit이나 Mockito 같은 테스트 프레임워크를 사용
- 테스트하기 좋은 코드를 작성

### 컨텍스트 캐싱

스프링은 기본적으로 애플리케이션 컨텍스트를 캐싱하는데, 동일한 설정을 사용하는 테스트는 동일한 컨텍스트를 재사용한다.

```java

@SpringBootTest
public class OrderServiceTest {
    // ...
}

@SpringBootTest
public class PaymentServiceTest {
    // ...
}
```

하지만 다른 설정을 사용하는 테스트는 새로운 컨텍스트를 생성하게 된다.

```java

@ActiveProfiles("test")
@SpringBootTest
public class OrderServiceTest {
    // ...
}

@ActiveProfiles("dev") // 다른 프로파일
@SpringBootTest
public class PaymentServiceTest {
    // ...
}
```

`TestContext` 프레임워크는 다음 구성 매개 변수를 사용하여 컨텍스트 캐싱을 하는데, 다를 경우 새로운 컨텍스트를 생성하게 된다.

| 항목                          | 관련 애노테이션/인자                       |
|-----------------------------|-----------------------------------|
| `locations`                 | `@ContextConfiguration` 애노테이션의 인자 |
| `classes`                   | `@ContextConfiguration` 애노테이션의 인자 |
| `contextInitializerClasses` | `@ContextConfiguration` 애노테이션의 인자 |
| `contextCustomizers`        | `ContextCustomizerFactory`에서 로드   |
| `contextLoader`             | `@ContextConfiguration` 애노테이션의 인자 |
| `parent`                    | `@ContextHierarchy`               |
| `activeProfiles`            | `@ActiveProfiles`                 |
| `propertySourceLocations`   | `@TestPropertySource`             |
| `propertySourceProperties`  | `@TestPropertySource`             |
| `resourceBasePath`          | `@WebAppConfiguration`            |

### 공통된 설정 사용

컨텍스트 캐싱을 이용하여 공통된 설정을 사용하면 컨텍스트를 재사용할 수 있어 테스트 실행 속도를 높일 수 있다.

```java
// 공통 설정 클래스
@SpringBootTest
@Transactional
public abstract class IntegrationTestSupport {
    // ...
}

// 공통 설정을 사용하는 테스트
public class OrderServiceTest extends IntegrationTestSupport {
    // ...
}
```

상속 받아 구현을 하면 공통된 설정을 사용할 수 있으며, 이를 통해 손쉽게 컨텍스트를 관리하고 실행 환경들을 통일 및 제어할 수 있다.
