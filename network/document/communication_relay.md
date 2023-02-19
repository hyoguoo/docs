# 통신 중계(Communication Relay)

웹 애플리케이션이 네트워크 상에서 통신을 할 때, 클라이언트와 서버 사이에 위치하여 클라이언트의 리퀘스트를 중계하는 프로그램을 통신 중계라고 한다.  
통신 중계를 해주는 이유와 종류는 프록시, 게이트웨이, 터널, 캐시 등이 있다.

## 프록시(Proxy)

> 클라이언트와 서버 사이에 위치하여 클라이언트의 리퀘스트를 중계하는 프로그램

- 클라이언트로부터 모든 요청을 받아 서버로 전달하고 서버(Origin)로부터 받은 응답을 클라이언트에 전달하는 역할
- 프록시 서버를 여러 대 경유하는 경우도 있으며 이를 체인 프록시(Chain Proxy)라고 함
- 사용 이유
    - 웹 트래픽 흐름 속에서 신뢰할 수 있는 중계자 역할을 하여 보안을 위해 사용
    - 특정 웹사이트에 대한 엑세스 제한 및 로그를 획득하기 위해 프록시 서버를 사용하는 경우도 있음

## 프락시 캐시(Proxy Cache)

> 원본 서버의 리소스를 캐시하여 클라이언트에게 제공하는 프록시 서버

- 캐시를 사용하여 리소스를 가진 서버에 엑세스를 하지 않고 바로 캐시된 리소스를 제공하여 트래픽을 줄이고, 빠른 응답을 제공할 수 있음
- 캐시된 리소스가 변경되었는지 확인하는 작업이 필요

## 게이트웨이(Gateway)

> 다른 서버(비 HTTP)들의 중개자로 동작하는 서버

- 프록시와 매우 유사하게 동작하지만, 게이트웨이의 경우에는 그 다음에 있는 서버가 HTTP 서버 이외의 서비스를 제공하는 서버가 됨
- 주로 HTTP 트래픽을 다른 프로토콜로 변환하기 위해 사용
  `클라이언트 <---> 게이트웨이 <---> HTTP 이외의 서버`
- 그 예로 HTTP/FTP 게이트웨이는 FTP URI에 대한 HTTP 요청을 받아들인 뒤, FTP 프로토콜을 이용해 문서를 가져와 HTTP 메시지로 변환한 뒤 클라이언트에게 보낸다.

## 터널(Tunnel)

> 두 커넥션 사이에서 날(raw) 데이터를 열어보지 않고 그대로 전달해주는 HTTP 애플리케이션

- HTTP 터널은 주로 비 HTTP 데이터를 하나 이상의 HTTP 연결을 통해 그대로 전송해주기 위해 사용
- 클라이언트는 SSL 같은 암호화 통신을 통해 서버와 안전하게 통신하기 위해 사용

###### 출처

- [HTTP 완벽 가이드](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=294437345)