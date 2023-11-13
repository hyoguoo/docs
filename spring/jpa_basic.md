---
layout: editorial
---

# JPA

> Java Persistence API의 약자로, 자바 진영의 ORM 기술 표준

- ORM(Object Relational Mapping)은 객체와 관계형 데이터베이스의 데이터를 자동으로 매핑(연결)해주는 것
- 표준 명세
    - 인터페이스의 모음
    - 구현한 구현체로 Hibernate, EclipseLink, DataNucleus 등 존재
    - 보통 Hibernate 사용

## JPA의 장점

- SQL 중심 -> 객체 중심
- 생산성
- 유지보수
- 패러다임의 불일치 해결
    - 상속
    - 연관관계
    - 객체 그래프 탐색
    - 비교
- 성능
- 데이터 접근 추상화와 벤더 독립성

## Configuration

JPA를 사용하기 위해서는 아래와 같은 설정이 필요하다.  
스프링 부트에서는 아래의 빈들이 자동적으로 설정되어 있기 때문에 별도의 직접 설정할 필요는 없다.

```java

@Configuration
public class JpaConfig {

    // 어떤 데이터베이스를 사용할 것인지 설정
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/jpa_basic?serverTimezone=UTC");
        dataSource.setUsername("root");
        dataSource.setPassword("password");

        return dataSource;
    }

    // JPA 구현체 설정
    @Bean
    public JpaVendorAdapter jpaVendorAdapter(JpaProperties jpaProperties) {
        HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();
        jpaVendorAdapter.setShowSql(jpaProperties.isShowSql());
        jpaVendorAdapter.setGenerateDdl(jpaProperties.isGenerateDdl());
        jpaVendorAdapter.setDatabasePlatform(jpaProperties.getDatabasePlatform());

        return jpaVendorAdapter;
    }

    // Entity 객체를 생성하고 관리해주는 EntityManagerFactory 설정
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter, JpaProperties jpaProperties) {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactoryBean.setDataSource(dataSource);
        entityManagerFactoryBean.setPackagesToScan("com.example.jpa_basic.domain");
        entityManagerFactoryBean.setJpaVendorAdapter(jpaVendorAdapter);

        Properties jpaProperties = new Properties();
        jpaProperties.putAll(jpaProperties.getProperties());
        entityManagerFactoryBean.setJpaProperties(jpaProperties);

        return entityManagerFactoryBean;
    }

    // Transaction을 관리해주는 TransactionManager 설정
    @Bean
    public PlatformTransactionManager transactionManager(LocalContainerEntityManagerFactoryBean entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory.getObject());

        return transactionManager;
    }
}
```

###### 참고자료

- [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic)