# Spring MVC(Model View Controller)

> Spring에서 제공하는 웹 모듈로, Model / View / Controller 세 가지 구성요소를 사용해 HTTP 요청을 처리를 지원해주는 프레임워크

## DispatcherServlet

`org.springframework.web.servlet.DispatcherServlet`

스프링 MVC의 핵심으로, 프론트 컨트롤러 역할을 한다.

- 부모 클래스에서 `HttpServlet`을 상속 받아 사용하고, 서블릿으로 동작
- 스프링 부트는 `DispatcherServlet`을 자동으로 등록하면서 모든 경로에 (`urlPatterns = "/"`) 매핑

## 요청 흐름

1. 서블릿이 호출되면 `HttpServlet`의 `service()` 메서드가 호출
2. 스프링 MVC의 `DispatcherServlet`은 `FrameworkServlet`의 `service()` 메서드를 오버라이딩이 되어있음
3. `FrameworkServlet`의 `service()` 메서드는 여러 메서드를 호출하는데, `doDispatch()` 메서드가 가장 중요한 메서드

### `doDispatch()` 코드 분석

```java
class Example {
    // ...
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        ModelAndView mv = null;
        // 1. 핸들러 조회
        mappedHandler = getHandler(processedRequest);
        if (mappedHandler == null) {
            noHandlerFound(processedRequest, response);
            return;
        }
        // 2.핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
        HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
        /**
         * 3. 핸들러 어댑터 실행
         * 4. 핸들러 어댑터를 통해 핸들러 실행
         * 5. ModelAndView 반환 mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
         */
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
        // 뷰 렌더링 호출
        render(mv, request, response);
    }

    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
        View view;
        String viewName = mv.getViewName(); //6. 뷰 리졸버를 통해서 뷰 찾기, 7.View 반환
        view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
        // 8. 뷰 렌더링
        view.render(mv.getModelInternal(), request, response);
    }
}
```

## Spring MVC Flow

![img.png](../image/spring_mvc_flow.png)

| 순서  |         내용         |                              설명                              |
|:---:|:------------------:|:------------------------------------------------------------:|
|  1  |       핸들러 조회       |                핸들러 매핑을 통해 요청 URL에 매핑된 핸들러 조회                 |
|  2  |     핸들러 어댑터 조회     |                   핸들러를 처리할 수 있는 핸들러 어댑터 조회                   |
|  3  |     핸들러 어댑터 실행     |                          핸들러 어댑터 실행                          |
|  4  | 핸들러 어댑터를 통해 핸들러 실행 |                      핸들러 어댑터를 통해 핸들러 실행                      |
|  5  |  ModelAndView 반환   |          핸들러 어댑터는 핸들러의 반환 정보를 ModelAndView로 변환해서 반환          |
|  6  |  뷰 리졸버를 통해서 뷰 찾기   |                       뷰 리졸버를 통해서 뷰를 찾음                       |
|  7  |      View 반환       | 뷰 리졸버는 논리 뷰 이름을 물리 뷰 이름으로 변환해서 반환 후 렌더링 역할을 담항하는 View 객체를 반환 |
|  8  |       뷰 렌더링        |             View 객체의 `render()` 메서드를 호출해서 뷰 렌더링              |

###### 출처

- https://www.inflearn.com/course/스프링-mvc-1