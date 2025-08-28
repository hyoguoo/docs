---
layout: editorial
---

# Spring Boot Auto Configuration(스프링 부트 자동 설정)

스프링 부트는 다양한 라이브러리와 기술 스택을 지원하며, 개발자가 별도의 코드 작성 없이도 쉽게 사용할 수 있도록 자동 설정(Auto Configuration) 기능을 제공한다.

## 자동 설정에서의 프로퍼티 적용 흐름(DataSource 예시)

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
