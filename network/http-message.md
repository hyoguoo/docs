---
layout: editorial
---

# HTTP Message

HTTP 메시지는 단순한 줄 단위의 문자열이고, 이진 형식이 아닌 일반 텍스트 형식이기 때문에 사람이 쉽게 읽을 수 있다.  
HTTP 메시지는 HTTP 애플리케이션 간에 주고 받는 데이터의 단위이며, HTTP 애플리케이션은 HTTP 메시지를 통해 요청과 응답을 주고 받는다.

## 메시지의 흐름

메시지는 항상 업스트림에서 다운스트림으로 흐르며, 서버냐 클라이언트냐를 나누는 개념이 아니고 메시지의 발송자와 수신자를 나누는 개념이다.

- 인바운드(Inbound) : 클라이언트에서 서버로의 방향
- 아웃바운드(Outbound) : 서버에서 클라이언트로의 방향
- 업스트림(Upstream) : 발송자
- 다운스트림(Downstream) : 수신자

## HTTP 구조

HTTP 메시지는 크게 아래 세 개로 구성되어 있다.

- Start Line: 메시지의 첫 줄로, 메시지의 종류와 버전 등과 무엇을 하는지에 대한 정보를 담고 있다.
- Headers: HTTP 전송에 필요한 모든 부가정보로, 0개 이상의 헤더 필드로 구성되어 있다.
- Message Body: 실제 전송할 데이터로, 필요에 따라 생기는 데이터이다.

각 줄은 CRLF(Carriage Return, Line Feed)로 끝나며, 각 부분은 CRLF로 구분된다.  
하지만 모든 HTTP 애플리케이션이 CRLF를 제대로 사용하고 있지 않기 때문에, 그냥 개행 문자도 받아들일 수 있는 HTTP 애플리케이션으로 개발하는 것이 좋다.

```http request
<start-line>
<headers>
<CRLF>
<message-body>
```

** CRLF: Carriage Return, Line Feed, 즉 `\r\n`을 의미하며 캐리지 리턴과 개행 문자로 구성된 문자열

### HTTP Request Message

```http request
<method> <request-URI> <HTTP-version>
<header>
<CRLF>
<entity-body>
```

위 형태를 가지고 있으며, 자세한 예시는 아래와 같다.

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

entity body는 생략될 수 있으며, 생략된 경우에는 CRLF로 끝나는 메시지가 된다.

### HTTP Response Message

```http request
<HTTP-version> <status-code> <reason-phrase>
<header>
<CRLF>
<entity-body>
```

위 형태를 가지고 있으며, 자세한 예시는 아래와 같다.

```http request
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

각 부분에 대한 자세한 설명은 아래와 같다.

### Start Line

HTTP 메시지의 첫 줄로, 요청과 응답에 따라 구성에 약간 차이가 있으며, 메시지의 종류와 버전 등과 무엇을 하는지에 대한 정보를 담고 있다.

- Request

```http request
<method> <request-URI> <HTTP-version>
```

- Response

```http request
<HTTP-version> <status-code> <reason-phrase>
```

### Headers

HTTP 전송에 필요한 모든 부가정보를 담고 있으며, 메시지 내용/크기/압축/인증 등을 포함한다.

```http request
status line
request header field(|| response header field)
general header field
entity header field
etc..
CRLF
```

HTTP 헤더는 크게 아래와 같이 구분할 수 있으며, 각 분류 안에 많은 헤더 필드가 존재한다.

- General Header: 메시지에 대한 기본적인 정보를 가진 헤더
- Request Header: 요청에 대한 정보, 요청자에 대한 정보나 어떤 리소스를 요청하는지에 대한 정보를 가진 헤더
- Response Header: 응답에 대한 정보, 응답자에 대한 정보나 응답에 대한 부가적인 정보를 가진 헤더
- Entity Header: 엔티티 바디에 대한 정보, 엔티티 바디의 데이터 타입이나 길이 등 엔티티 바디에 대한 부가적인 정보를 가진 헤더
- Extension Header: 명세에 정의되지 않은 새로운 헤더, 사용자가 직접 만들어 사용한 헤더

### Message Body(Entity Body)

실제 전송할 데이터로, byte로 표현할 수 있는 모든 데이터를 전송할 수 있다.  
모든 메시지가 가지고 있지는 않으며, 그냥 CRLF로 끝나는 메시지도 존재한다.

###### 참고자료

- [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-웹-네트워크)
- [(그림으로 배우는) http & network basic](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=9788931447897&isbn=9788931447897&cipId=200443691%2C)
- [HTTP 완벽 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=HTTP+완벽+가이드&isbn=9788966261208&cipId=200309770%2C4096969)
