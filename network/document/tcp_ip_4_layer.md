# TCP/IP 4 Layer

> 네트워크의 기본으로 인터넷을 포함하여 일반적으로 사용하고 있는 네트워크는 TCP/IP라는 프로토콜에서 움직이며 HTTP도 그 중 하나

## TCP/IP 4 계층

|        프로토콜 계층        |        예시        |                       설명                        |
|:---------------------:|:----------------:|:-----------------------------------------------:|
| 애플리케이션 계층 - HTTP, FTP | 웹 브라우저, 채팅 프로그램  |       유저에게 제공되는 애플리케이션에서 사용하는 통신의 움직임을 결정       |
|   전송 계층 - TCP, UDP    |   OS(TCP/UDP)    | 애플리케이션 계층에 네트워크로 접속되어 있는 2대의 컴퓨터 사이의 데이터 흐름을 제공 |
|      인터넷 계층 - IP      |      OS(IP)      |     네트워크 상에서 패킷의 이동을 다룸(패킷: 전송하는 데이터 최소 단위)     |
|     네트워크 인터페이스 계층     | LAN 드라이버, LAN 장비 |               네트워크에 접속하는 하드웨어적인 면               |

## 통신의 흐름 예시

1. 애플리케이션 계층의 `Socket Library`를 통해 OS에 메시지 전달
2. OS의 TCP 계층에서 메시지(데이터)에 TCP 정보 생성
3. OS의 IP 계층에서 TCP 정보에 IP 정보 생성
4. 네트워크 인터페이스 LAN 카드를 통해 나갈 때 `Ethernet Frame`을 통해 전송

## 전송된 데이터 형태

```
-- IP 패킷: 출발지 IP, 목적지 IP, 기타
------ TCP 세그먼트: 출발지 포트, 목적지 포트, 전송 제어, 순서, 검증 정보, 기타
---------- 데이터: 전송 데이터
```

# 웹 브라우저 요청 흐름

1. DNS 서버에 도메인 이름을 IP 주소로 변환
2. HTTP 레이어에서 웹 서버에 보낼 HTTP 요청 메시지 작성

```http request
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

3. SOCKET 라이브러리를 통해 TCP/IP 연결
4. TCP 레이어에서 HTTP 요청 메시지를 통신하기 쉽도록 패킷으로 분해(TCP 세그먼트로 작성)

```http request
출발지 IP, PORT + 목적지 IP, PORT
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

5. IP 레이어에서 상대가 어디에 있는지 찾아 중계해가며 서버로 패킷 전달
6. 서버에서 요청 패킷을 받아 HTTP 요청 메시지 파싱
7. 서버에서 HTTP 응답 메시지 생성

```http request
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
...

<!doctype html>
<html itemscope="" itemtype="http://schema.org/SearchResultsPage" lang="ko">
<head>
<meta charset="UTF-8">
<title>hello - Google 검색</title>
```

8. 응답 패킷 전송
9. 응답 패킷 도착
10. 웹 브라우저 HTML 렌더링

###### 출처

- https://www.inflearn.com/course/http-웹-네트워크