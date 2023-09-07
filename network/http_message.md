---
layout: editorial
---

# HTTP Message

> HTTP 애플레키에션 간에 주고받는 데이터 단위

## 메시지의 흐름

- 인바운드(Inbound) : 클라이언트에서 서버로의 방향
- 아웃바운드(Outbound) : 서버에서 클라이언트로의 방향
- 업스트림(Upstream) : 발송자
- 다운스트림(Downstream) : 수신자

메시지는 항상 업스트림에서 다운스트림으로 흐르며, 서버냐 클라이언트냐를 나누는 개념이 아니고 메시지의 발송자와 수신자를 나누는 개념이다.

## HTTP 구조

```http request
<start-line>
<headers>
<CRLF>
<message-body>
```

** CRLF: Carriage Return Line Feed, 즉 `\r\n`을 의미하며 쉽게 말해 줄바꿈을 의미

- HTTP Request Message 구조 예시

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

- HTTP Response Message 구조 예시

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

```http request
<method> <request-URI> <HTTP-version>
```

- Response

```http request
<HTTP-version> <status-code> <reason-phrase>
```

HTTP 메시지의 첫 줄로, 메시지의 종류와 버전 등과 무엇을 하는지에 대한 정보를 담고 있다.

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
    - Extension Header: 명세에 정의되지 않은 새로운 헤더

### Message Body(Entity Body)

실제 전송할 데이터로, byte로 표현할 수 있는 모든 데이터를 전송할 수 있다.  
기본적으로 Message Body와 Entity Body는 동일하지만 청크 전송 인코딩을 사용하면 Message Body와 Entity Body가 다를 수 있다.

- Message Body: HTTP 통신의 기본단위로 Octet sequence로 구성되며 통신을 통해 전송
- Entity Body: 리퀘스트랑 리스폰스의 페이로드로 전송되는 정보로 Entity Header + Entity Body로 구성, 요청이나 응답에서 전달할 실제 데이터
    - Entity Header: Entity Body에 대한 부가 정보로, 엔티티 본문의 데이터를 해석할 수 있는 정보 제공
    - RFC 7230에서는 Entity Body라는 용어를 사용하지 않고, 표현(Representation)이라는 용어를 사용한다.

## 표현(Representation)

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

###### 참고자료

- [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-웹-네트워크)
- [(그림으로 배우는) http & network basic](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=9788931447897&isbn=9788931447897&cipId=200443691%2C)
- [HTTP 완벽 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=HTTP+완벽+가이드&isbn=9788966261208&cipId=200309770%2C4096969)