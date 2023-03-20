# View Resolver

Spring MVC 프레임워크에서 ViewResolver는 컨트롤러 메서드에 의해 반환된 논리적 뷰 이름을 클라이언트에 대한 응답을 렌더링하는 데 사용할 수 있는 실제 뷰 구현으로 해석하는 역할

## ViewResolver 종류

|         ViewResolver         |               설명               |
|:----------------------------:|:------------------------------:|
| InternalResourceViewResolver | view 이름을 웹 애플리케이션의 JSP 페이지에 매핑 |
|    ThymeleafViewResolver     |   view 이름을 Thymeleaf 템플릿에 매핑   |

이외에도 UrlBasedViewResolver, FreeMarkerViewResolver 등이 있음

뷰 이름을 확인할 때 ViewResolver는 일반적으로 다음 단계를 수행합니다.

## ViewResolver 동작 과정

1. 핸들러 어댑터를 통해 컨트롤러 메서드가 반환한 논리적 뷰 이름을 확인
   1-1. 뷰 이름에 추가해야 하는 접두사 및/또는 접미사가 있는지 확인
2. 뷰 이름으로 ViewResolver를 순서대로 호출
3. ViewResolver가 뷰 이름을 처리할 수 있는지 확인
   3-1. 처리할 수 있다면 해당 뷰를 반환
   3-2. 처리할 수 없다면 다음 ViewResolver를 호출
   3-3. 모든 ViewResolver가 처리할 수 없다면 예외 발생
4. ViewResolver가 반환한 View를 사용하여 뷰 렌더링
5. 뷰 렌더링 결과를 클라이언트에게 응답

###### 출처

- https://www.inflearn.com/course/스프링-mvc-1