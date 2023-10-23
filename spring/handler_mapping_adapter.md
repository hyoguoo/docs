---
layout: editorial
---

# HandlerMapping & HandlerAdapter(핸들러 매핑 & 핸들러 어댑터)

HandlerMapping 및 HandlerAdapter는 들어오는 HTTP 요청을 처리하고 적절한 Controller 메서드에 매핑하는 MVC 프레임워크의 중요한 역할을 한다.  
HTTP 요청을 처리하기 위한 유연하고 확장 가능한 메커니즘을 제공한다.

## HandlerMapping

들어오는 요청의 URL을 분석하여 적절한 컨트롤러(처리할 수 있는 핸들러) 메서드를 찾아 매핑하는 역할을 한다.

### 구현체와 우선 순위

HandlerMapping은 여러 구현체가 있지만, 스프링 MVC에서는 주로 다음 두 가지를 사용되며 우선순위는 다음과 같다.

|             구현체              |                                          설명                                           | 우선 순위 |
|:----------------------------:|:-------------------------------------------------------------------------------------:|------:|
| RequestMappingHandlerMapping | @Controller, @RequestMapping, @GetMapping 등 요청 URL을 처리할 컨트롤러 Annotation 중 일치하는 핸들러 매핑 |     0 |
|  BeanNameUrlHandlerMapping   |                           스프링 빈의 이름 중 요청 URL과 일치하는 핸들러를 매핑                            |     1 |

## HandlerAdapter

요청을 컨트롤러 메서드에 적용하는 역할로 HandlerMapping이 찾아낸 컨트롤러를 실행하는 역할을 한다.  
컨트롤러 메서드를 래핑하고 요청을 처리하는 데 필요한 컨텍스트와 매개 변수를 제공하고, 실제 컨트롤러 메서드를 호출하여 반환된 결과를 ModelAndView로 변환한다.

### 구현체와 우선 순위

HandlerAdapter는 대표적으로 아래의 구현체가 있으며, 스프링 MVC에서는 주로 다음 세 가지를 사용되며 우선순위는 다음과 같다.

|              구현체               |                   설명                    | 우선 순위 |
|:------------------------------:|:---------------------------------------:|------:|
|  RequestMappingHandlerAdapter  |  애노테이션 기반의 컨트롤러인 @RequestMapping 에서 사용  |     0 |
|   HttpRequestHandlerAdapter    | HttpRequestHandler 인터페이스를 구현한 컨트롤러에서 사용 |     1 |
| SimpleControllerHandlerAdapter |     Controller 인터페이스를 구현한 컨트롤러에서 사용     |     2 |

###### 참고자료

- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/스프링-mvc-1)