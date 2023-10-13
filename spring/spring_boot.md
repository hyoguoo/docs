---
layout: editorial
---

# 스프링 부트

과거 스프링 프레임워크는 직접 설정을 해야하고 구성하는 등의 번거로움이 상당히 많았지만 스프링 부트가 등장하면서 손쉽게 웹 애플리케이션을 만들 수 있게 되었다.  
스프링 부트는 스프링 프레임워크의 설정과 기능을 단순화하고 자동화하여 개발자가 더 쉽게 스프링 기반 애플리케이션을 개발할 수 있도록 도와준다.

- 라이브러리 버전 관리 자동화
    - 여러 라이브러리의 버전을 명시하지 않더라도 적합한 버전을 자동으로 관리
- AutoConfig 설정 자동화
    - 다양한 공통적인 설정을 자동으로 구성
- 내장 웹서버 제공
    - 톰캣과 같은 별도 웹서버를 설치하지 않아도 내장 웹서버를 통해 웹 애플리케이션을 실행할 수 있도록 지원
- 실행 가능한 JAR
    - 순서 자바 애플리케이션 프로그램을 실행하는 것처럼 실행에 대한 편리성 제공

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

여기서 `@SpringBootApplication` 의 핵심 어노테이션은 아래 세 개의 어노테이션이라고 볼 수 있다.

- `@SpringBootConfiguration`
    - `@Configuration`의 하위 어노테이션으로 `@Configuration`과 동일한 역할 수행
    - 동일한 역할을 수행하지만 스프링 부트에서 해당 어노테이션을 기준으로 설정을 불러오기 위해 사용함
    - 스프링 부트의 설정을 나타내는 어노테이션
- `@EnableAutoConfiguration`
    - 스프링 부트의 자동 설정을 활성화하는 어노테이션
    - 해당 어노테이션 내 `@Import(AutoConfigurationImportSelector.class)` 코드가 `META-INF/.../...imports` 파일을 읽어 자동 설정을 수행
- `@ComponentScan`
    - `@SpringBootApplication` 어노테이션이 선언된 클래스의 패키지와 하위 패키지에서 스프링 컴포넌트를 검색하고 등록하는 역할
    - 때문에 `@SpringBootApplication` 어노테이션은 최상위 패키지에 위치하는 것이 좋음(최초 생성시 기본적으로 최상위 패키지에 위치)

## SpringBootApplication

위의 Main 코드를 보면 `SpringApplication.run(Application.class, args);` 라는 코드가 있다.  
해당 코드는 `SpringApplication`의 `run` 정적 메서드를 호출하는 것으로, 애플리케이션을 실행하는 역할을 수행한다.  
해당 메서드는 `ConfigurableApplicationContext` 타입의 `run` 메서드를 호출하는데, 아래의 역할을 수행한다.

- 스프링 애플리케이션을 초기화하고 실행하는 주요한 역할을 수행
- 실행 리스너들에게 이벤트 발생
- 자동 구성을 수행하고 환경 설정 준비
- 애플리케이션 컨텍스트 생성 및 초기화

## 스프링 부트의 실행 과정

위 내용을 바탕으로 스프링 부트의 실행 과정을 정리하면 아래와 같다.

1. @SpringBootApplication 어노테이션이 선언된 메인 클래스에서 main 메서드 실행
2. SpringApplication.run(Application.class, args) 메서드를 호출하여 스프링 애플리케이션 실행
3. SpringApplication은 애플리케이션 컨텍스트를 생성 및 초기화
4. 내장 웹 서버가 시작됩니다.
5. @EnableAutoConfiguration 어노테이션을 통해 자동 구성 수향
6. 클래스 패스와 설정 정보를 기반으로 필요한 빈(bean)들이 자동 생성 및 등록
7. @ComponentScan 어노테이션을 통해 컴포넌트 스캔이 수행되어 스프링 컴포넌트 등록
8. 애플리케이션 실행 이벤트를 처리하는 리스너들이 호출
9. 애플리케이션의 초기화가 완료되면 내장 웹 서버가 요청을 처리할 수 있는 상태가 됨
10. 애플리케이션 실행 완료

###### 참고자료

- [스프링 부트 - 핵심 원리와 활용](https://www.inflearn.com/course/스프링부트-핵심원리-활용)
- [망나니개발자 티스토리](https://mangkyu.tistory.com/213)