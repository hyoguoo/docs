# 스프링 웹 개발 기초
1. 정적 컨텐츠
2. MVC & 템플릿 엔진
3. API

## 정적 컨텐츠
![img.png](../image/spring_static_contents_operating.png)

## MVC & 템플릿 엔진
![img.png](../image/spring_view_page_operating.png)

## API
![img.png](../image/spring_response_body_operating.png)
- http body에 문자 내용 반환
- `HttpMessageConverter` 동작
  - 문자: `StringHttpMessageConverter`
  - 객체: `MappingJackson2HttpMessageConverter`
  - 그 외에도 여러 converter 존재