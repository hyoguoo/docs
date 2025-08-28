---
layout: editorial
---

# Spring Boot(스프링 부트)

과거 스프링 프레임워크는 직접 설정을 해야하고 구성하는 등의 번거로움이 상당히 많았지만 스프링 부트가 등장하면서 손쉽게 웹 애플리케이션을 만들 수 있게 되었다.

- 라이브러리 버전 관리 자동화
    - 여러 라이브러리의 버전을 명시하지 않더라도 적합한 버전을 자동으로 관리
- 자동 설정 자동화
    - 다양한 공통적인 설정을 자동으로 구성
- 내장 웹 서버 제공
    - 톰캣과 같은 별도 웹 서버를 설치하지 않아도 내장 웹 서버를 통해 웹 애플리케이션을 실행할 수 있도록 지원
- 실행 가능한 JAR
    - 순수 자바 애플리케이션 프로그램을 실행하는 것처럼 실행에 대한 편리성 제공

이처럼 스프링 부트는 스프링 프레임워크의 설정과 기능을 단순화하고 자동화하여 개발자가 더 쉽게 스프링 기반 애플리케이션을 개발할 수 있도록 도와준다.

## @SpringBootApplication

Spring Initializr를 통해 프로젝트를 생성하면 아래와 같이 `@SpringBootApplication` 어노테이션이 선언된 클래스가 생성된다.

```java

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

실제로 해당 애노테이션을 살펴보면 아래와 같이 정의된 것을 볼 수 있다.

```java

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)})
public @interface SpringBootApplication {

    // ...
}
```

### 핵심 구성 요소

1. @SpringBootConfiguration
    - @Configuration의 메타 애노테이션
    - 스프링 부트의 애플리케이션 설정 클래스임을 나타냄
2. @EnableAutoConfiguration
    - 자동 설정 활성화
    - @Import를 통해 `META-INF/.../...imports` 파일의 자동 설정 클래스를 읽어 등록
3. @ComponentScan
    - 현재 패키지와 하위 패키지를 스캔하여 컴포넌트를 빈으로 등록
    - 제외/포함 필터로 세밀 제어 가능

## SpringBootApplication 클래스

Main 코드에서 `SpringApplication.run(Application.class, args);`를 통해 `run` 메서드를 호출하는데, 아래의 역할을 수행하게 된다.

- 스프링 애플리케이션을 초기화하고 실행하는 주요한 역할을 수행
- 실행 리스너들에게 이벤트 발생
- 자동 구성을 수행하고 환경 설정 준비
- 애플리케이션 컨텍스트 생성 및 초기화
- ApplicationRunner와 CommandLineRunner 실행

## 스프링 부트의 실행 과정

위 내용을 바탕으로 스프링 부트의 실행 과정을 정리하면 아래와 같다.

1. @SpringBootApplication이 선언된 메인 클래스에서 main 실행
2. SpringApplication.run 호출로 환경(Environment) 준비, 프로파일 적용
3. ApplicationContext 생성 및 초기화, ApplicationContextInitializer 적용
4. @ComponentScan으로 애플리케이션 컴포넌트 등록
5. @EnableAutoConfiguration에 의해 자동 설정 클래스 로딩 및 조건 평가
6. 컨텍스트 refresh, 빈 후처리기와 라이프사이클 콜백 실행
7. 웹 애플리케이션이면 내장 웹 서버(Tomcat/Jetty/Undertow) 시작
8. ApplicationRunner, CommandLineRunner 실행
9. 애플리케이션 준비 완료, 요청 처리 가능 상태

## 외부 설정과 프로필

스프링 부트는 외부 설정을 `application.properties` 또는 `application.yml` 파일을 사용하는 관리할 수 있는데, 프로필(Profile)을 통해 환경별로 다른 설정을 적용할 수 있다.

```yaml
# 단일 파일에서 프로필별 설정 관리 예시 (application.yml)
# 기본 설정
server:
  port: 8080

app:
  name: demo
  message: 기본 메시지

---
# 프로필이 local일 때 설정
spring:
  config:
    activate:
      on-profile: local
app:
  message: 로컬 메시지
server:
  port: 8081

---
# 프로필이 prod일 때 설정
spring:
  config:
    activate:
      on-profile: prod
app:
  message: 운영 메시지
server:
  port: 8080
```

```yaml
# 별도 파일에서 프로필별 설정 관리 예시
# (application-prod.yml)
server:
  - port: 8080
app:
  - message: 운영 메시지
# (application-local.yml)
server:
  - port: 8081
app:
  - message: 로컬 메시지
```

- `application.yml` 안에서 `---`로 구분하거나 별도 파일로 관리 가능
    - `spring.config.activate.on-profile`로 프로필 지정
- `application-{profile}.yml` 형식으로 별도 파일 관리 가능

### 프로필 실행 방법

- JAR 실행: `java -jar build/libs/app.jar --spring.profiles.active=local`
- Gradle: `./gradlew bootRun --args='--spring.profiles.active=local'`
- Maven: `./mvnw spring-boot:run -Dspring-boot.run.profiles=local`

## 프로퍼티 값 주입 방법

```yaml
# application.yml
app:
  api:
    base-url: [ https://api.example.com ](https://api.example.com)
    key: ${APP_API_KEY}   # 환경 변수로 외부화(예: export APP_API_KEY=...)
    timeout: 3s           # Duration
    retry-max: 5          # int
    endpoints: # List<String>
      - users
      - payments
      - reports
    default-headers: # Map<String, String>
      X-CLIENT: demo
      X-TRACK: enabled
    max-body-size: 10MB   # DataSize
    feature-enabled: true # boolean
````

### Environment.getProperty

```java

@Component
public class EnvReader {

    private final URI baseUrl;
    private final int retryMax;
    private final Duration timeout;
    private final DataSize maxBodySize;

    public EnvReader(Environment env) {
        // getProperty 메서드를 사용하여 타입 변환 및 기본값 설정
        this.baseUrl = env.getProperty("app.api.base-url", URI.class);
        this.retryMax = env.getProperty("app.api.retry-max", Integer.class, 3);
        this.timeout = env.getProperty("app.api.timeout", Duration.class, Duration.ofSeconds(2));
        this.maxBodySize = env.getProperty("app.api.max-body-size", DataSize.class, DataSize.ofMegabytes(5));
    }
}
```

- `Environment` 빈을 주입받아 `getProperty()` 메서드로 값을 조회
- 프로퍼티 키를 문자열로 직접 지정(기본값 설정 가능)
- `Duration`, `DataSize` 등 다양한 타입으로 변환을 지원

### @Value

```java

@Component
public class ValueReader {

    @Value("${app.api.key}")
    private String key;
    @Value("${app.api.timeout}")
    private Duration timeout;
    @Value("${app.api.retry-max:3}")
    private int retryMax;
    @Value("${app.api.endpoints}")
    private List<String> endpoints;
    @Value("${app.api.max-body-size}")
    private DataSize maxBodySize;

}
```

- `${...}` 플레이스홀더를 사용해 프로퍼티 키를 명시
- SpEL(Spring Expression Language)을 활용한 동적인 값 주입 가능
- 타입 변환 지원
- `:` 문자를 이용해 기본값 설정

간편하게 사용 가능하지만 다음과 같은 단점이 있다.

- 관련 프로퍼티들이 코드 전반에 흩어져 응집도가 낮아질 수 있음
- 타입 안정성이 낮고, 컴파일 시점 검증이 어려움

### @ConfigurationProperties

```java
// ApiProperties.java, 설정 프로퍼티 클래스 정의
@ConfigurationProperties(prefix = "app.api")
@Validated // jakarta.validation.constraints.* 어노테이션을 사용한 검증 활성화
public record ApiProperties(
                @NotEmpty
                URI baseUrl,
                String key,
                Duration timeout,
                int retryMax,
                List<String> endpoints,
                Map<String, String> defaultHeaders,
                DataSize maxBodySize,
                boolean featureEnabled
        ) {

}
```

메인 애플리케이션 클래스에 어노테이션을 추가하여 `@ConfigurationProperties`가 붙은 클래스를 자동으로 스캔하고 빈으로 등록하도록 설정하는 과정이 필요하다.

```java
// Application.java
@SpringBootApplication
@ConfigurationPropertiesScan // "com.example.demo" 패키지 하위를 스캔
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

빈으로 등록되어있기 때문에 필요한 곳에 주입하여 사용할 수 있다.

```java
// PropsReader.java
@Component
public class PropsReader {

    private final ApiProperties properties;

    public PropsReader(ApiProperties properties) {
        this.properties = properties;
    }
}
```

- 관련 프로퍼티를 구조화된 객체로 관리하여 가독성과 유지보수성 향상
- `@Validated`와 `jakarta.validation` 어노테이션을 통해 애플리케이션 시작 시점에 프로퍼티 값 검증 가능
- 실제 프로퍼티 파일 대신 객체를 직접 생성하여 주입이 가능하여 테스트 용이성 향상

### 방법별 비교 요약

| 구분                         | 용도 및 특징                           | 장점                       | 단점                       |
|----------------------------|-----------------------------------|--------------------------|--------------------------|
| `Environment.getProperty`  | 프레임워크 내부 또는 동적으로 키를 결정해야 할 때 사용   | 스프링 컨테이너 어디서든 주입받아 사용 가능 | 키를 문자열로 관리, 타입 안정성 부족    |
| `@Value`                   | 소수의, 서로 관련 없는 단일 값을 주입할 때 간편하게 사용 | SpEL 지원, 사용법이 직관적        | 프로퍼티 관리 분산, 컴파일 시점 검증 불가 |
| `@ConfigurationProperties` | 여러 관련 값을 하나의 그룹으로 묶어 관리할 때 사용     | 타입 안전, 강력한 검증 기능, 테스트 용이 | 별도의 클래스 정의 필요            |

## 자동 설정에서의 프로퍼티 사용 방법(DataSource 예시)

개발자는 의존성을 추가하고 `application.yml`에 프로퍼티를 작성하는 것만으로 복잡한 설정 및 빈 등록을 자동으로 완료할 수 있게되는데, 그 과정은 아래와 같다.

### 1. @EnableAutoConfiguration의 역할

`@SpringBootApplication`에 포함된 `@EnableAutoConfiguration` 애노테이션이 스프링 부트의 자동 설정 기능을 활성화한다.

- 내부적으로 `@Import(AutoConfigurationImportSelector.class)`를 사용하여 자동 설정 클래스를 스프링 컨테이너에 등록
- `spring-boot-autoconfigure` 라이브러리 내의 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일 참조
- 해당 파일에 정의된 자동 설정 클래스들을 로드할 후보로 지정

### 2. 조건부 설정(@Conditional)

로드된 모든 자동 설정 클래스가 실제로 동작하는 것은 아니며, 각각의 자동 설정 클래스는 `@Conditional...` 애노테이션 그룹을 통해 특정 조건이 만족될 때만 활성화된다.

### 3. 프로퍼티 바인딩(@ConfigurationProperties)

조건들이 모두 충족되면, `DataSourceAutoConfiguration`은 활성되고, 필요한 설정 값들을 읽어오게 된다.

```java
// DataSourceProperties.java (실제 스프링 부트 내부 클래스)

@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {

    private String url;
    private String username;
    private String password;
    private String driverClassName;

    // ... getter, setter ...
}
```

`@ConfigurationProperties(prefix = "spring.datasource")` 애노테이션을 통해 `spring.datasource` 값들이 이 객체의 필드에 자동으로 바인딩된다.

### 4. 빈(Bean) 생성 및 등록

자동 설정 클래스(`DataSourceAutoConfiguration`)는 값이 채워진 `DataSourceProperties` 객체를 사용하여 `DataSource` 빈을 생성하고 스프링 컨테이너에 등록하게 된다.

```java
abstract class DataSourceConfiguration {

    // DataSource 생성을 담당하는 정적 메서드
    // JdbcConnectionDetails 로부터 URL, 계정 정보 등을 꺼내와 DataSourceBuilder에 전달
    @SuppressWarnings("unchecked")
    private static <T> T createDataSource(JdbcConnectionDetails connectionDetails, Class<? extends DataSource> type,
            ClassLoader classLoader) {
        return (T) DataSourceBuilder.create(classLoader)
                .type(type)
                .driverClassName(connectionDetails.getDriverClassName())
                .url(connectionDetails.getJdbcUrl())
                .username(connectionDetails.getUsername())
                .password(connectionDetails.getPassword())
                .build();
    }

    // ...

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(HikariDataSource.class)
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.HikariDataSource",
            matchIfMissing = true)
    static class Hikari {

        @Bean
        static HikariJdbcConnectionDetailsBeanPostProcessor jdbcConnectionDetailsHikariBeanPostProcessor(
                ObjectProvider<JdbcConnectionDetails> connectionDetailsProvider) {
            return new HikariJdbcConnectionDetailsBeanPostProcessor(connectionDetailsProvider);
        }

        @Bean
        @ConfigurationProperties(prefix = "spring.datasource.hikari")
        HikariDataSource dataSource(DataSourceProperties properties, JdbcConnectionDetails connectionDetails) {
            HikariDataSource dataSource = createDataSource(connectionDetails, HikariDataSource.class,
                    properties.getClassLoader());
            if (StringUtils.hasText(properties.getName())) {
                dataSource.setPoolName(properties.getName());
            }
            return dataSource;
        }

    }
```

`createDataSource` 메서드를 통해 `DataSource` 객체를 생성하는데, 해당 생성 인자인 `JdbcConnectionDetails` 인터페이스를 구현체에 연결 정보가 담겨 연결 정보를 제공한다.

```java
final class PropertiesJdbcConnectionDetails implements JdbcConnectionDetails {

    private final DataSourceProperties properties;

    PropertiesJdbcConnectionDetails(DataSourceProperties properties) {
        this.properties = properties;
    }

    @Override
    public String getUsername() {
        return this.properties.determineUsername();
    }

    @Override
    public String getPassword() {
        return this.properties.determinePassword();
    }

    @Override
    public String getJdbcUrl() {
        return this.properties.determineUrl();
    }

    @Override
    public String getDriverClassName() {
        return this.properties.determineDriverClassName();
    }

    @Override
    public String getXaDataSourceClassName() {
        return (this.properties.getXa().getDataSourceClassName() != null)
                ? this.properties.getXa().getDataSourceClassName()
                : JdbcConnectionDetails.super.getXaDataSourceClassName();
    }

}
```

###### 참고자료

- [스프링 부트 - 핵심 원리와 활용](https://www.inflearn.com/course/스프링부트-핵심원리-활용)
- [망나니개발자 티스토리](https://mangkyu.tistory.com/213)