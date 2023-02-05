# HTTP(HyperText Transfer Protocol)

## HTTP 특징

- `Client` <---> `Server` 존재
- `request`를 보내면 `response`로 되돌아옴
- `request` 없이는 `response` 없음
- 상태를 유지하지 않는 프로토콜(`stateless`)
- `GET` / `POST` / `PUT` / `HEAD` / `DELETE` / `OPTIONS` 메서드 존재

## 연결 방식

|              연결 방법               |                    연결 방식                     | 연결 종료 방법 |
|:--------------------------------:|:--------------------------------------------:|:--------:|
| 개별 연결(Non-Persistent Connection) |             연결 후 요청 후 응답 후 연결 종료             |  연결 종료   |
|   지속 연결(Persistent Connection)   |             연결 후 요청 후 응답 후 연결 유지             | 연결 종료 명시 |
|        파이프라인화(Pipelining)        | 연결 후 요청 후 응답 후 연결 유지, 리스폰스를 기다리지 않고 다음 요청 가능 | 연결 종료 명시 |

초기 HTTP 에서는 통신 한 번 할때마다 TCP에 의해 연결이 되고 종료됐으나 많은 데이터를 처리할 때 매번 TCP 연결을 하고 종료하는 것은 비효율적  
이를 해결하기 위해 명시적으로 연결을 종료하지 않는 이상 TCP 연결을 유지하는 방법을 사용하게 됐다.

## 쿠키(Cookie)

`Stateful`는 리소스 사용을 줄이는 장점이 있지만, `Stateless`는 `Client`의 상태를 유지하지 않기 때문에 `Client`의 상태를 유지하기 위해 `Cookie`를 사용한다.

## HTTP Message

> HTTP 애플레키에션 간에 주고받는 데이터 단위

복수 행(개행 문자 CR+LF)의 데이터로 구성되며 헤더와 행바디로 구분됨

- Request Header

```http request
request line
request header field
general header field
entity header field
etc..
CRLF
```

- Response Header

```http request
status line
response header field
general header field
entity header field
etc..
CRLF
```

### Message Body / Entity Body

기본적으로 Message Body와 Entity Body는 동일하지만 전송 코딩이 적용되어 있을 경우 Entity Body의 내용이 변화할 수 있기 때문에 Message Body와 달라짐

- Message Body: HTTP 통신의 기본단위로 Octet sequence로 구성되며 통신을 통해 전송
- Entity Body: 리퀘스트랑 리스폰스의 페이로드로 전송되는 정보로 Entity Header Field + Entity Body로 구성

## Encoding

|      인코딩 방식      |                                            설명                                            |
|:----------------:|:----------------------------------------------------------------------------------------:|
| Content-Encoding |                        전송 데이터의 압축 방식을 지정하고, 수신 측에서는 압축을 해제하여 디코딩                         |
| Chunked Encoding | 엔티티 바디를 청크로 분해하여 청크 사이즈를 16진수로 사용하여 단락을 표시하고 엔티티 바디 끝에 0(CR+LF)를 기록, 수신 측에서는 청크를 합쳐서 디코딩 |

HTTP 전송 시 그대로 전송하는 방법도 있지만 인코딩을 적용하여 전송 효율을 높일 수 있으나 리소스를 사용하게 됨

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

## Range Requests

엔티티 범위를 지정하여 다운로드할 수 있어야 했고 그 범위를 지정하여 요청하는 것

- request example

```http request
GET /bigfile HTTP/1.1
Host: www.example.com
Range: bytes=1000-1999
```

- response example
  - 206 상태 코드와 함께 Content-Range 헤더 필드를 포함
  - 만약 서버에서 리퀘스트 레인지를 지원하지 않는 경우 200 상태 코드와 함께 전체 엔티티를 전송

```http response
HTTP/1.1 206 Partial Content
Date: Mon, 27 Jul 2009 12:28:53 GMT
Content-Type: text/plain
Content-Range: bytes 1000-1999/67589
Content-Length: 1000
```

## Content Negotiation

- 클라이언트가 서버에게 자신이 원하는 콘텐츠의 종류를 요청하는 것
- `request message`에 Accept 헤더 필드를 사용하여 요청

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

###### 출처

- https://www.inflearn.com/course/http-웹-네트워크
- [그림으로 배우는 HTTP & Network Basic](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=51908132)