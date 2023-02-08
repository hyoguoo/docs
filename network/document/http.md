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

## 연결 방식

|              연결 방법               |                    연결 방식                     | 연결 종료 방법 |
|:--------------------------------:|:--------------------------------------------:|:--------:|
| 개별 연결(Non-Persistent Connection) |             연결 후 요청 후 응답 후 연결 종료             |  연결 종료   |
|   지속 연결(Persistent Connection)   |             연결 후 요청 후 응답 후 연결 유지             | 연결 종료 명시 |
|        파이프라인화(Pipelining)        | 연결 후 요청 후 응답 후 연결 유지, 리스폰스를 기다리지 않고 다음 요청 가능 | 연결 종료 명시 |

초기 HTTP 에서는 통신 한 번 할때마다 TCP에 의해 연결이 되고 종료됐으나 많은 데이터를 처리할 때 매번 TCP 연결을 하고 종료하는 것은 비효율적  
이를 해결하기 위해 명시적으로 연결을 종료하지 않는 이상 TCP 연결을 유지하는 방법을 사용하게 됐다.

## HTTP Message

> HTTP 애플레키에션 간에 주고받는 데이터 단위

```http request
<start-line>
<headers>
<CRLF>
<message-body>
```

### HTTP Request Message 구조 예시

```http request
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:70.0) Gecko/20100101 Firefox/70.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: ko-KR,ko;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

### HTTP Response Message 구조 예시

```http response
HTTP/1.1 200 OK
Date: Mon, 18 Nov 2019 07:28:00 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Mon, 18 Nov 2019 07:27:30 GMT
ETag: "1d3-5a115e6ee8400"
Accept-Ranges: bytes
Content-Length: 467
Vary: Accept-Encoding
Content-Type: text/html

<html>
  ...
</html>
```

### Start Line

- Request
    - HTTP 메서드
    - 요청 대상
    - HTTP 버전
- Response
    - HTTP 버전
    - 상태 코드
    - 이유 문구

### Headers

```http request
status line
request header field(|| response header field)
general header field
entity header field
etc..
CRLF
```

- HTTP 전송에 필요한 모든 부가정보
    - 메시지 바디의 내용/바디의 크기/압축/인증 등 표준 헤더 필더가 매우 많으며, 사용자 정의 헤더 필드도 사용 가능
- HTTP 헤더는 크게 아래와 같이 구분
    - General Header: 메시지 전체에 적용되는 정보
    - Request(Response) Header: 요청(응답)에 대한 정보
    - Entity Header: 엔티티 바디에 대한 정보

### Message Body(Entity Body)

실제 전송할 데이터로, byte로 표현할 수 있는 모든 데이터를 전송할 수 있다.  
기본적으로 Message Body와 Entity Body는 동일하지만 청크 전송 인코딩을 사용하면 Message Body와 Entity Body가 다를 수 있다.

- Message Body: HTTP 통신의 기본단위로 Octet sequence로 구성되며 통신을 통해 전송
- Entity Body: 리퀘스트랑 리스폰스의 페이로드로 전송되는 정보로 Entity Header + Entity Body로 구성, 요청이나 응답에서 전달할 실제 데이터
    - Entity Header: Entity Body에 대한 부가 정보로, 엔티티 본문의 데이터를 해석할 수 있는 정보 제공
    - RFC 7230에서는 Entity Body라는 용어를 사용하지 않고, 표현(Representation)이라는 용어를 사용한다.

### 표현(`Representation`)

- Representation Metadata: 표현 데이터를 해석할 수 있는 정보
    - 형식/인코딩/문자집합과 같은 데이터 표현에 대한 정보
    - 클라이언트 소프트웨어에서 데이터를 올바르게 처리하는 데 사용
    - 일반적으로 HTTP 헤더 또는 표현 데이터 자체에 포함
- Representation Data: 표현 데이터
    - 웹페이지의 HTML 컨텐츠와 같이 전송되는 실제 페이로드 또는 데이터
    - HTTP 메시지 본문에 포함
- Representation Header: 표현 메타데이터를 전달하는 헤더
    - 컨텐츠 유형, 컨텐츠 길이 및 인코딩과 같이 요청되는 리소스 또는 반환되는 응답에 대한 정보를 제공하는 HTTP 헤더의 구성 요소
    - 클라이언트와 서버에서 보내고 받는 표현 유형을 협상하는 데 사용

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

## 쿠키(Cookie)

`Stateful`는 리소스 사용을 줄이는 장점이 있지만, `Stateless`는 `Client`의 상태를 유지하지 않기 때문에 `Client`의 상태를 유지하기 위해 `Cookie`를 사용한다.

- 사용처
    - 사용자 로그인 세션 관리
    - 사용자 선호 설정
    - 광고 정보 트래킹
- 항상 `HTTP` 헤더에 포함되어 전송됨
    - 네트워크 트래픽 증가
    - 최소한의 정보만을 담는 것을 권장하며, 보안에 민감한 정보는 저장하지 않는 것이 좋음

### 쿠키의 생명주기

- `Session Cookie`: 브라우저가 종료되면 쿠키가 삭제됨
- `Persistent Cookie`: 브라우저가 종료되어도 쿠키가 삭제되지 않음
- `Expires`나 `Max-Age`를 지정하지 않으면 세션 쿠키로 생성됨
    - `Expires` : 쿠키의 만료 시간을 지정
    - `Max-Age` : 쿠키의 만료 시간을 초 단위로 지정

### 도메인

해당 쿠키가 적용될 도메인을 지정하는 속성(`domain=example.com`)

- 명시하는 경우
    - `Domain` 속성에 도메인을 지정
    - `Domain` 속성에 지정한 도메인과 하위 도메인에 쿠키가 적용됨(`dev.example.com`에도 적용됨)
- 명시하지 않는 경우
    - `Domain` 속성에 현재 도메인이 지정됨
    - `Domain` 속성에 지정한 도메인만 쿠키가 적용됨(`dev.example.com`에는 적용되지 않음)

### 경로

해당 쿠키가 적용될 하위 경로를 지정하는 속성으로 일반적으로 루트로 지정(`path=/`)

### 보안

- `Secure`
    - 쿠키는 원래 `HTTP`, `HTTPS`를 구분하지 않고 전송
    - `Secure` 속성을 지정하면 `HTTPS`에서만 쿠키가 전송됨
- `HttpOnly`
    - 자바스크립트 코드에서 쿠키에 접근이 가능한데 이를 막기 위해 `HttpOnly` 속성을 지정(XSS 공격 방지)
    - HTTP 전송에만 쿠키를 사용할 수 있게 됨
- `SameSite`
    - `SameSite` 속성을 지정하면 `SameSite` 속성에 지정한 도메인과 같은 도메인에서만 쿠키가 전송됨
    - `SameSite=Strict` : 동일한 도메인에서만 쿠키가 전송됨

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

###### 출처

- https://www.inflearn.com/course/http-웹-네트워크
- [그림으로 배우는 HTTP & Network Basic](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=51908132)