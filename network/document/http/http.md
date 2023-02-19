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

## 캐시(Cache)

> 웹 서버에서 클라이언트로 전송된 리소스를 임시로 저장하는 장소

```http request
HTTP/1.1 200 OK
Content-Type: image/jpeg
Cache-Control: max-age=3600

ghuhgrhejiofjdwqieuqwiloguoguioqijddwhud
```

`Cache-Control` 헤더를 사용하여 캐시를 제어할 수 있으며 `max-age` 속성을 사용하여 캐시 유효 시간을 지정할 수 있다.  
유효기간 내에 재요청이 발생하면 캐시된 리소스를 사용하고 유효기간이 지나면 서버에 재요청을 보내 새로운 리소스를 받아온다.  
여기서 리소스가 변경되지 않았을 경우 불필요한 네트워크 트래픽이 발생하는데, 이를 방지하기 위해 검증 헤더와 조건부 요청을 사용한다.

- Cache-Control 캐시 지시어
    - `max-age` : 캐시 유효 시간을 초 단위로 지정
    - `no-cache` : 데이터는 캐시해도 되지만, 캐시된 데이터를 사용하기 전에 항상 원 서버에 검증을 요청
    - `no-store` : 데이터에 민감한 정보가 포함되어 있으므로 캐시에 저장하지 않음
    - `must-revalidate`: 캐시 만료 후 최조 조회 시 원 서버에 검증 필요
    - `expires` : 캐시 유효 시간을 날짜와 시간으로 지정, `max-age`가 지정되어 있으면 무시되며 `max-age` 사용을 권장`
    - `public` : 응답 데이터를 public 캐시(프록시 캐시 서버)에 저장할 수 있음
    - `private` : 응답 데이터를 private 캐시에 저장할 수 있음(해당 사용자만 캐시를 사용할 수 있음, 기본값)
    - `s-maxage` : `max-age`와 동일하지만, 프록시 캐시에만 적용됨

### 검증 헤더 & 조건부 요청

- 조건부 요청 헤더(`If-Modified-Since`, `If-None-Match`): 클라이언트가 캐시된 리소스를 사용할 수 있는지 검증하기 위해 서버에게 요청하는 헤더

```http request
GET /image.jpg HTTP/1.1
if-Modified-Since: Thu, 09 Feb 2023 05:59:59 KST
if-None-Match: "oguoguogu"
```

- 검증 헤더(`Last-Modified`, `ETag`): 서버에서 클라이언트에게 리소스의 변경 여부를 알려주는 헤더

```http request
HTTP/1.1 200 OK
Content-Type: image/jpeg
Cache-Control: max-age=3600
Last-Modified: Thu, 09 Feb 2023 05:59:59 KST
ETag: "oguoguogu"
```

#### `Last-Modified` & `If-Modified-Since` 방식

`Last-Modified` 리소스가 마지막으로 변경된 시간을 나타내는 헤더로 리소스를 요청할 때 `If-Modified-Since` 헤더와 함께 요청한다.

- 수정된 경우
  서버에서 리소스 수정일을 확인하고 변경되지 않았다면 HTTP Body를 제외하여 `304 Not Modified` 응답을 보내게 되고,
  서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신하며 캐시된 리소스를 사용하게 된다.

- 변경된 경우
  `200 OK` 응답을 보내고 새로운 리소스를 전송한다.

여기서 데이터를 수정해서 저장했지만, 실제로 내용이 변경되지 않았을 경우에도 `Last-Modified` 헤더의 값이 변경되기 때문에 캐시된 리소스를 사용할 수 없는 단점이 존재한다.

#### `ETag(Entity Tag)` & `If-None-Match` 방식

`ETag`는 리소스의 고유한 버전 정보를 나타내는 헤더로, 리소스를 요청할 때 `If-None-Match` 헤더와 함께 요청한다.  
`ETag`와 `If-None-Match` 헤더를 비교하여 리소스의 변경 여부를 확인하게 되고 로직은 위의 방식과 동일하다.

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

###### 출처

- https://www.inflearn.com/course/http-웹-네트워크
- [그림으로 배우는 HTTP & Network Basic](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=51908132)