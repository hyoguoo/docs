---
layout: editorial
---

# 엔티티(Entity)

## 메시지와 엔티티

HTTP 메시지 - 컨테이너, HTTP 엔티티 - 화물 비유로 설명할 수 있으며 실제 HTTP 메시지를 예시로 들면 아래와 같다

```http request
HTTP/1.0 200 OK
Server: Netscape-Enterprise/3.6
Date: Sun, 17 Sep 2000 00:01:05 GMT
# 아래 줄부터 마지막 줄까지 엔티티
Content-Type: text/plain # 엔티티 헤더
Content-Length: 18 # 엔티티 헤더

Hi! I'm a message! # 엔티티 본문
```

- 엔티티 헤더
    - HTTP/1.1 기준으로 위의 두 가지 정보 말고 10가지 주요 헤더 필드들이 존재한다.
    - (Content-Type, Content-Length, Content-Language, Content-Encoding, Content-Location, Content-Range, Content-MD5,
      Last-Modified, Expires, Allow, ETag, Cache-Control)

- 엔티티 본문
    - 가공되지 않은 데이터를 담고 있으며, 다른 정보들은 헤더에 담겨있다.
    - 가공되지 않은 날 데이터이기 때문에 헤더에 데이터의 의미에 대해 설명하는 정보들이 담겨있다.

## Content-Length

본문의 크기를 바이트 단위로 나타내며, 청크 인코딩이 아닌 이상 필수 헤더 필드이다.  
HTTP 보안을 강화하고나 압축하기 위해 콘텐츠 인코딩을 사용할 수 있는데, 이 때 Content-Length 헤더는 인코딩 된 본문의 크기를 나타낸다.  
해당 헤더는 단순히 크기를 나타내는 것이지만 통신에 있어서 중요한 역할을 할 수 있게된다.

- 잘림 검출: Content-Length 헤더를 통해 본문의 크기를 알 수 있기 때문에, 메시지를 받는 도중 본문이 잘려서 전송되었는지 알 수 있다.
- 지속 커넥션(Persistent Connection): Content-Length 헤더를 통해 본문의 크기를 알 수 있어 메시지가 끝난 지점을 알 수 있다.

## 엔티티 요약

HTTP는 TCP/IP와 같이 신뢰할 수 있는 전송 프로토콜 위에서 구현되지만, 여러 변수들로 인해 전송 중 변형되는 일이 발생할 수 있다.(불완전한 트랜스코딩 프락시 or 버그 많은 중개자 프락시 등)
엔티티 본문이 의도치 않은 변경을 감지하기 위해, 최초 엔티티가 생성될 때 간단한 체크섬을 생성하여, 그 체크섬으로 기본적인 검사를 할 수 있다.

## 미디어 타입과 차셋(Charset)

Content-Type 헤더 필드는 엔티티 본문의 MIME 타입을 기술하며, 표현은 `type/subtype` 형태로 되어있다. 

- type: text, image, audio, video, application, message, multipart
- subtype: type에 따라 맞는 하위 타입이 존재
- 

###### 참고자료

- [HTTP 완벽 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=HTTP+완벽+가이드&isbn=9788966261208&cipId=200309770%2C4096969)