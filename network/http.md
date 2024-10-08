---
layout: editorial
---

# HTTP(HyperText Transfer Protocol)

HTML/TEXT, IMAGE, 음성, 영상, 파일, JSON, XML 등 다양한 데이터를 전송 가능한 프로토콜로, 현재 네트워크 통신에서 많이 사용되는 프로토콜이다.  
주로 서버와 클라이언트 사이에서 요청/응답을 주고받는데 사용되고 있다.

## 역사

- HTTP/0.9: 초기 버전(GET 메서드만 지원 / 헤더 없음)
- HTTP/1.0: 메서드와 헤더를 추가로 지원하는 버전(지속 연결 미지원)
- HTTP/1.1: 현재 가장 널리 사용되는 버전으로, 지속 연결, 파이프라이닝(pipelining)을 지원하여 성능 향상
- HTTP/2: HTTP/1.1의 성능 문제를 보완하기 위해 설계된 버전으로, 헤더 압축, 스트림 다중화 등으로 성능을 개선
- HTTP/3: TCP 대신 UDP 프로토콜 위에서 작동하며, 기존 TCP 기반의 문제점을 개선하여 더 빠른 성능 제공

## 전송 계층과 HTTP

- TCP: HTTP/1.1, HTTP/2
- UDP: HTTP/3

HTTP/3는 UDP 프로토콜 위에 애플리케이션 레벨에서 성능을 최적화하도록 새로 설계된 프로토콜이다.
([참고 링크](https://evan-moon.github.io/2019/10/08/what-is-http3/))

## HTTP 특징

- 요청-응답 기반의 `Client`-`Server` 구조를 가짐
- `request`를 보내면 `response`로 돌아오며, `request` 없이는 `response`가 존재하지 않음
- 상태를 유지하지 않음(`stateless`)
- HTMP / 이미지 / 영상 / 파일 / JSON / XML 등 다양한 데이터를 전송 가능(미디어 독립적)
- 비연결성(`connectionless`)
- 지속 연결(`Keep-Alive`)을 통해 하나의 TCP 커넥션 여러 개의 요청과 응답을 처리(Three-way Handshake를 한 번만 하면 되기 때문에 성능 향상)
- `GET` / `POST` / `PUT` / `HEAD` / `DELETE` / `OPTIONS` 메서드 존재

## 전송 방식

HTTP는 다양한 방식으로 데이터를 전송하는데, 이 방식은 메시지 전송 시 사용하는 헤더에 따라 결정된다.

|           전송 방식            |       사용 헤더       |           설명           |
|:--------------------------:|:-----------------:|:----------------------:|
|  단순 전송(Simplest Transfer)  |  Content-Length   |    메시지 바디의 크기를 알려줌     |
| 압축 전송(Compressed Transfer) | Content-Encoding  |  메시지 바디를 압축한 방식을 알려줌   |
|  분할 전송(Chunked Transfer)   | Transfer-Encoding | 메시지 바디를 여러 조각으로 나누어 전송 |
|   범위 전송(Ranged Transfer)   |   Content-Range   |     메시지 바디의 일부만 요청     |

## Content Negotiation

클라이언트가 서버에게 어떤 컨텐츠를 원하는지 알려주는 기능으로, 서버는 이 정보를 통해 가장 적절한 컨텐츠를 제공할 수 있다.

- 클라이언트는 `request message`에 Accept* 헤더 필드를 사용하여 요청
- 서버는 `response message`에 Content* 헤더 필드를 사용하여 응답
- 언어 태그(`Accept-Language`) / 문자셋(`Accept-Charset`) / 인코딩(`Accept-Encoding`) / 미디어 타입(`Accept`) 등을 사용
- 0 ~ 1 사이의 Quality Value(QValue)를 사용하여 우선순위를 나타냄(`Accept-Language: ko-KR, en-US;q=0.9, en;q=0.8`)

## Status Code

`request`에 대한 서버의 응답 상태를 나타내는 세 자리 숫자 코드로 구성되어 있으며, 응답 상태를 크게 5가지로 분류할 수 있다.

| code |    class     |     description      |
|:----:|:------------:|:--------------------:|
| 1xx  | information  |   리퀘스트를 받아들여 처리 중    |
| 2xx  |   success    |      리퀘스트 정상 처리      |
| 3xx  | redirection  | 리퀘스트를 완료하려면 추가 행동 필요 |
| 4xx  | client error |     리퀘스트 이해 불가능      |
| 5xx  | server error |    서버가 리퀘스트 처리 실패    |

자세한 내용은 [HTTP Status](https://developer.mozilla.org/ko/docs/Web/HTTP/Status) 참고

## MIME(Multipurpose Internet Mail Extensions)

텍스트/영상/이미지 등 다양한 데이터를 다루기 위한 기능으로 `Content-Type` 헤더 필드를 사용하여 데이터의 종류를 나타낸다.  
웹 서버에서 MIME 타입을 지정하면 웹 브라우저는 다룰 수 있는 객체인지 확인하고, 해당 타입에 맞는 애플리케이션을 실행하여 데이터를 처리한다.

- 이미지 등의 바이너리 데이터를 아스키(ASCII) 문자열에 인코딩하는 방법과 데이터 종류를 나타내는 방법 등을 규정
- 확장 사양에 있는 멀티파트(Multipart)라고 하는 여러 다른 종류의 데이터를 수용하는 방법 사용
- `HTTP`도 멀티파트에 대응하고 있어 하나의 메시지 바디 내부에 여러 엔티티를 포함시킬 수 있음
- 전송 방식
    - multipart/form-data: Web 폼으로부터 파일 업로드에 사용
    - multipart/byteranges: 상태 코드 206(Partial Content) `response message`가 복수 범위의 내용을 포함할 때 사용

###### 참고자료

- [HTTP 완벽 가이드](https://kobic.net/book/bookInfo/view.do?isbn=9788966261208)
- [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-웹-네트워크)
- [그림으로 배우는 HTTP & Network Basic](https://kobic.net/book/bookInfo/view.do?isbn=9788931447897)
