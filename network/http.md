---
layout: editorial
---

# HTTP(HyperText Transfer Protocol)

HTML/TEXT, IMAGE, 음성, 영상, 파일, JSON, XML 등 다양한 데이터를 전송 가능하다.  
서버 간 통신을 할 때도 대부분 HTTP를 통해 통신한다.

## 역사

- HTTP/0.9: GET 메서드만 지원, 헤더 없음
- HTTP/1.0: 메서드/헤더 추가
- HTTP/1.1: 가장 많이 사용하는 버전, 지속 연결 지원, 파이프라이닝 지원
- HTTP/2: HTTP/1.1의 단점 보완, 성능 향상
- HTTP/3: TCP 대신 UDP 사용, 성능 향상

## 기반 프로토콜

- TCP: HTTP/1.1, HTTP/2
- UDP: HTTP/3

HTTP/3는 UDP 프로토콜 위에 애플리케이션 레벨에서 성능을 최적화하도록 새로 설계된
프로토콜이다.([참고 링크](https://evan-moon.github.io/2019/10/08/what-is-http3/))

## HTTP 특징

- `Client` <---> `Server` 존재
- `request`를 보내면 `response`로 되돌아옴
- `request` 없이는 `response` 없음
- 상태를 유지하지 않는 프로토콜(`stateless`)
- 비연결성(`connectionless`)
- `GET` / `POST` / `PUT` / `HEAD` / `DELETE` / `OPTIONS` 메서드 존재

## 전송 방식

|           전송 방식            |       사용 헤더       |           설명           |
|:--------------------------:|:-----------------:|:----------------------:|
|  단순 전송(Simplest Transfer)  |  Content-Length   |    메시지 바디의 크기를 알려줌     |
| 압축 전송(Compressed Transfer) | Content-Encoding  |  메시지 바디를 압축한 방식을 알려줌   |
|  분할 전송(Chunked Transfer)   | Transfer-Encoding | 메시지 바디를 여러 조각으로 나누어 전송 |
|   범위 전송(Ranged Transfer)   |   Content-Range   |     메시지 바디의 일부만 요청     |

## Content Negotiation

- 클라이언트가 서버에게 자신이 원하는 콘텐츠의 종류를 요청하는 것
- `request message`에 Accept 헤더 필드를 사용하여 요청
- 우선순위를 지정하여 요청할 수 있음

## Status Code

> `request`에 대한 서버의 응답 상태를 나타내는 세 자리 숫자 코드

| code |    class     |     description      |
|:----:|:------------:|:--------------------:|
| 1xx  | information  |   리퀘스트를 받아들여 처리 중    |
| 2xx  |   success    |      리퀘스트 정상 처리      |
| 3xx  | redirection  | 리퀘스트를 완료하려면 추가 행동 필요 |
| 4xx  | client error |     리퀘스트 이해 불가능      |
| 5xx  | server error |    서버가 리퀘스트 처리 실패    |

자세한 내용은 [HTTP Status](https://developer.mozilla.org/ko/docs/Web/HTTP/Status) 참고

## MIME(Multipurpose Internet Mail Extensions)

> 텍스트/영상/이미지 등 다양한 데이터를 다루기 위한 기능

- 이미지 등의 바이너리 데이터를 아스키(ASCII) 문자열에 인코딩하는 방법과 데이터 종류를 나타내는 방법 등을 규정
- 확장 사양에 있는 멀티파트(Multipart)라고 하는 여러 다른 종류의 데이터를 수용하는 방법 사용
- `HTTP`도 멀티파트에 대응하고 있어 하나의 메시지 바디 내부에 여러 엔티티를 포함시킬 수 있음
- 전송 방식
    - multipart/form-data: Web 폼으로부터 파일 업로드에 사용
    - multipart/byteranges: 상태 코드 206(Partial Content) `response message`가 복수 범위의 내용을 포함할 때 사용

HTTP 메시지로 멀티파트를 사용할 경우 Content-Type 헤더 필드를 사용  
멀티파트 각각 엔티티를 구분하기 위해 각각의 엔티티에 고유한 경계(boundary) 문자열을 부여

## 보안상 문제가 발생하는 HTTP

HTTP에는 많은 장점들이 있지만 아래와 같은 문제점들이 존재한다.

### 평문 통신(메시지 탈취 가능)

TCP/IP 특성상 중간에 누군가가 패킷을 가로채면 내용을 확인할 수 있으며, HTTP는 암호화가 되어 있지 않기 때문에 패킷을 가로채면 내용을 확인할 수 있다.  
이를 해결하기 위해 암호화를 사용하는 방법이 있다.

1. 통신 암호화

통신을 암호화 하는 방법으로 SSL(Secure Socket Layer)와 TLS(Transport Layer Security)를 사용해 암호화를 할 수 있다.

2. 컨텐츠 암호화

통신을 통해 전달하고 있는 메시지를 암호화 하는 방법으로, 암호화된 메시지를 전달하고 받는다.  
클라이언트/서버 모두 암호화/복호화 과정을 거쳐야 하기 때문에 암호화/복호화 과정이 추가되어 성능이 떨어지는 단점이 있으며, 주로 웹 서비스에서 사용할 수 있다.

### 통신 상대 확인 불가능(위장 가능)

HTTP는 클라이언트가 서버에게 요청을 보낼 때, 서버가 해당 클라이언트가 맞는지 확인하지 않기 때문에 위장이 가능하다.  
이를 해결하기 위해 증명서를 사용하는 방법이 있다.

- 증명서

앞서 설명한 SSL를 사용하면 클라이언트와 서버가 서로를 확인할 수 있다.  
증명서는 서버나 클라이언트가 아닌 제 3자가 발급하는 인증서이며, 위조하는 것은 기술적으로 어렵기 때문에 위장을 방지할 수 있다.

### 완전성 확인 불가능(변조 가능)

HTTP는 메시지가 중간에 변조되지 않았는지 확인할 수 없기 때문에 변조가 가능하다.(요청이나 응답을 가로채서 변조하는 공격을 `중간자(Man in the Middle) 공격`이라고 한다.)  
이를 방지하기 위한 방법이 `디지털 서명`이라는 방법이 존재하나, 이 방법 하나만으로는 완벽한 보안을 보장할 수 없다.

###### 참고자료

- [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-웹-네트워크)
- [(그림으로 배우는) http & network basic](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=9788931447897&isbn=9788931447897&cipId=200443691%2C)