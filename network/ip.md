# IP - Internet Protocol

> 네트워크 상에서 컴퓨터들이 서로 통신하기 위해 사용하는 프로토콜

네트워크 층에 해당되며 IP 주소와는 다른 개념

## 역할

- 지정한 IP 주소(네트워크 상에서 컴퓨터를 식별하기 위한 주소)에 데이터 전달
- 패킷을 전송하는 역할
    - IP 패킷: 전송데이터를 포함한, 출발지 IP/목적지 IP 외 기타 정보가 들어있는 정보

## IP 프로토콜의 한계

- 비연결성 : 패킷을 받을 대상이 없거나 서비스 불능 상태여도 그대로 패킷 전송
- 비신뢰성 : 중간에 패킷이 사라지거나 패킷이 순서대로 안 오는 경우가 존재
- 프로그램 구분 : 같은 Ip를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상일 경우 구분이 불가능

이러한 한계를 보완하기 위해 [TCP](tcp.md) 프로토콜이 등장

###### 참고자료

- [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-웹-네트워크)
- [(그림으로 배우는) http & network basic](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=9788931447897&isbn=9788931447897&cipId=200443691%2C)