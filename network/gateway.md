---
layout: editorial
---

# Gateway(게이트웨이)

> 다른 서버(비 HTTP)들의 중개자로 동작하는 서버

- 프록시와 매우 유사하게 동작하지만, 게이트웨이의 경우에는 그 다음에 있는 서버가 HTTP 서버 이외의 서비스를 제공하는 서버가 됨
- 주로 HTTP 트래픽을 다른 프로토콜로 변환하기 위해 사용
  `클라이언트 <---> 게이트웨이 <---> HTTP 이외의 서버`
- 그 예로 HTTP/FTP 게이트웨이는 FTP URI에 대한 HTTP 요청을 받아들인 뒤, FTP 프로토콜을 이용해 문서를 가져와 HTTP 메시지로 변환한 뒤 클라이언트에게 보낸다.

###### 참고자료

- [HTTP 완벽 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=HTTP+완벽+가이드&isbn=9788966261208&cipId=200309770%2C4096969)
