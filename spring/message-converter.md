---
layout: editorial
---

# Message Converter(메시지 컨버터)

뷰 템플릿이나 정적 컨텐츠를 응답하는 것이 아니라, JSON, XML 또는 사용자 지정 형식과 같은 다양한 형식의 데이터를 교환하는 작업이 필요할 때 메시지 컨버터를 사용할 수 있다.
Spring MVC에서 다음의 경우 메시지 컨버터를 사용한다.

- HTTP 요청: `@RequestBody`, `HttpEntity`, `RequestEntity`
- HTTP 응답: `@ResponseBody`, `HttpEntity`, `ResponseEntity`

## HTTP 메시지 컨버터 인터페이스

`org.springframework.http.converter.HttpMessageConverter`

```java
package org.springframework.http.converter;

public interface HttpMessageConverter<T> {

    boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

    boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

    List<MediaType> getSupportedMediaTypes();

    T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
            throws IOException, HttpMessageNotReadableException;

    void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
            throws IOException, HttpMessageNotWritableException;
}
```

- `canRead()` / `canWrite()`: 메시지 컨버터가 해당 클래스, 미디어 타입을 지원하는지 확인
- `read()` / `write()`: 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

## 스프링 부트 기본 메시지 컨버터

스프링 부트에서는 다양한 메시지 컨버터를 제공하는데, 클래스 타입과 미디어 타입을 체크하여 사용여부를 체크하게 된다.  
우선 순위는 아래와 같으며 만족하지 않으면 다음 우선 순위의 메시지 컨버터를 사용한다.

1. `ByteArrayHttpMessageConverter`: `byte[]` 데이터 처리
2. `StringHttpMessageConverter`: `String` 문자로 데이터 처리
3. `MappingJackson2HttpMessageConverter`: `application/json` 처리

## HTTP 요청 데이터 읽기 및 응답 데이터 생성 과정

1. 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 `canRead()` 호출
    - 대상 클래스 타입을 지원하는지, HTTP 요청의 Content-Type 미디어 타입을 지원하는지 확인
2. `canRead()` 조건을 만족하는 경우 `read()` 호출하여 객체를 생성하고 반환
3. 컨트롤러 로직 실행
4. 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 `canWrite()` 호출
    - 대상 클래스 타입을 지원하는지, HTTP 응답의 Content-Type 미디어 타입을 지원하는지 확인
5. `canWrite()` 조건을 만족하는 경우 `write()` 호출하여 HTTP 응답 메시지 바디에 데이터 생성

###### 참고자료

- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/스프링-mvc-1)