---
layout: editorial
---

# View Resolver(뷰 리졸버)

Spring MVC 프레임워크에서 ViewResolver는 컨트롤러 메서드에 의해 반환된 논리적 뷰 이름을 클라이언트에 대한 응답을 렌더링하는 데 사용할 수 있는 실제 뷰 구현으로 해석하는 역할을 한다.

## ViewResolver 종류

|         ViewResolver         |               설명               |
|:----------------------------:|:------------------------------:|
| InternalResourceViewResolver | view 이름을 웹 애플리케이션의 JSP 페이지에 매핑 |
|    ThymeleafViewResolver     |   view 이름을 Thymeleaf 템플릿에 매핑   |

이외에도 UrlBasedViewResolver, FreeMarkerViewResolver 등 다양한 ViewResolver 구현체가 존재한다.

## prefix / suffix

ViewResolver는 논리적 뷰 이름에 prefix와 suffix를 추가하여 뷰를 찾을 수 있도록 처리할 수 있다.

```text
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

## ViewResolver 동작 과정

1. 핸들러 어댑터를 통해 컨트롤러 메서드가 반환한 논리적 뷰 이름을 확인
2. 뷰 이름으로 ViewResolver를 순서대로 호출
3. ViewResolver가 뷰 이름을 처리할 수 있는지 확인
    - 처리할 수 있다면 해당 뷰를 반환
    - 처리할 수 없다면 다음 ViewResolver를 호출
    - 모든 ViewResolver가 처리할 수 없다면 예외 발생
4. ViewResolver가 반환한 View를 사용하여 뷰 렌더링
5. 뷰 렌더링 결과를 클라이언트에게 응답

###### 참고자료

- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/스프링-mvc-1)