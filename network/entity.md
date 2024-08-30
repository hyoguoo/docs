---
layout: editorial
---

# Entity(엔티티)

## 메시지와 엔티티

HTTP 메시지 - 컨테이너, HTTP 엔티티 - 실질적인 화물 비유로 설명할 수 있으며 실제 HTTP 메시지를 예시로 들면 아래와 같다.

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
    - HTTP/1.1 기준 위의 두 가지 정보 포함하여 10가지 주요 헤더 필드 존재
    - 엔티티 본문의 메타데이터를 담고 있으며, 본문의 내용 / 인코딩 / 언어 등 관련된 정보 제공
    - Content-Type, Content-Length, Content-Language, Content-Encoding, Content-Location,
      Content-Range, Content-MD5, Last-Modified, Expires, Allow, ETag, Cache-Control
- 엔티티 본문
    - 실제 전송하려는 가공되지 않은 데이터 존재
    - 헤더 필드의 끝을 의미하는 빈 `CRLF` 바로 다음부터 본문 시작

## Content-Length

본문의 크기를 바이틀 단위로 나타내는 Content-Length 헤더는 HTTP 통신에서 특히 중요한 역할을 하기 때문에 대부분 상황에서 필수 헤더 필드이다.  
HTTP 보안을 강화하고나 압축하기 위해 콘텐츠 인코딩을 사용할 수 있는데, 이 때 Content-Length 헤더는 원문이 아닌 인코딩 된 본문의 크기를 나타낸다.  
본문을 갖는 것이 허용되지 않은 상황(HEAD 응답, 1xx/204/304 응답)에서는 Content-Length 무시된다.

### Content-Length 헤더의 중요성

해당 헤더는 단순히 크기를 나타내는 것 이상으로 통신에서 아래와 같은 중요한 역할을 한다.

- 잘림 검출: 본문의 크기와 비교해 본문을 받는 도중 잘림이 발생했는지 확인 가능
- 지속 커넥션: 본문의 크기를 통해 메시지가 끝난 지점을 알 수 있어, 지속 커넥션 사용 중 HTTP 메시지 경계를 명확하게 식별 가능

## 엔티티 요약

HTTP는 TCP/IP와 같이 신뢰할 수 있는 전송 프로토콜 위에서 구현되지만, 아래와 같은 이유들로 전송 중 메시지 일부분이 변형되는 일이 발생할 수 있다.

- 불완전한 트랜스코딩 프락시
    - 트랜스코딩 프락시는 대역폭 절약이나 클라이언트 호환을 위해 메시지 일부를 변경할 수 있는데, 이 과정에서 원본 데이터가 올바르지 않게 변형될 수 있음
- 버그 많은 중개자 프락시
    - 소프트웨어 버그 / 잘못된 설정 / 부적절한 보안 정책으로 데이터 손상 가능성 존재

엔티티 본문이 의도치 않은 변경을 감지하기 위해, 최초 엔티티가 생성될 때 간단한 체크섬을 생성하여, 그 체크섬으로 기본적인 검사를하여 엔티티가 변형되었는지 확인할 수 있다.

## 미디어 타입과 차셋(Charset)

Content-Type 헤더 필드는 엔티티 본문의 MIME 타입을 기술하며, 표현은 `type/subtype` 형태로 되어있다.

- type: text, image, audio, video, application, message, multipart
- subtype: type에 따라 맞는 하위 타입이 존재

###### 참고자료

- [HTTP 완벽 가이드](https://kobic.net/book/bookInfo/view.do?isbn=9788966261208)
