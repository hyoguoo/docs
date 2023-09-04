---
layout: editorial
---

# 전송 계층 (Transport Layer) - TCP/IP Layer 3

전송 계층은 응용 계층(TCP/IP L4)의 애플리케이션 프로세스 식별과, 네트워크 계층(TCP/IP L2)의 신뢰성/연결성 확립(네트워크 계층의 한계 극복)을 담당한다.

- 애플리케이션 프로세스 식별 -> 포트 번호
- 신뢰성/연결성 확립 -> TCP 프로토콜

## PORT

네트워크 내의 단일 장치에서 실행되는 프로세스를 식별하는 논리적인 단위이다.  
포트 번호는 16비트의 숫자로, 0 ~ 65535까지 사용할 수 있으며, 범위에 따라 아래와 같이 분류된다.

|            포트 종류             |      범위       |                     설명                      |
|:----------------------------:|:-------------:|:-------------------------------------------:|
| Well-known port(System Port) |   0 ~ 1023    |     잘 알려진 포트로, 특정 프로토콜에 할당된 포트(특정 프로토콜)     |
|       Registered port        | 1024 ~ 49151  | 위의 포트보다는 덜 범용적이지만 흔히 사용되는 포트(특정 기업의 애플리케이션) |
|         Dynamic port         | 49152 ~ 65535 |       사용자가 자유롭게 할당 가능한 포트(크롬 같은 브라우저)       |

IANA(Internet Assigned Numbers Authority)에서 관리하고 아래 링크에서 확인할 수 있다.  
[IANA](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)

## TCP(Transmission Control Protocol)

TCP는 전송 계층의 대표적인 프로토콜로, 신뢰성 있는 데이터 전송을 보장한다.  
TCP의 데이터 단위는 세그먼트(Segment)이며, 세그먼트는 TCP 헤더와 Payload(애플리케이션 계층에서 전달받은 데이터)로 구성된다.

### TCP Segment 헤더 구조

- Source Port: 송신지 포트 번호
- Destination Port: 수신지 포트 번호
- Sequence Number: 송수신되는 세그먼트 데이터 첫 바이트에 부여되는 순서 번호
- Acknowledgement Number: 순서 번호에 대한 응답(다음으로 수신받길 기대하는 바이트 번호)
- Control Bits: TCP 연결 설정/해제, 데이터 전송 등을 제어하는 비트
    - ACK: 세그먼트 승인을 나타내는 비트
    - SYN: 연결 수립을 위한 비트
    - FIN: 연결을 끝내기 위한 비트
    - 그 외: URG / PSH / RST ...
- Window Size: 수신지 윈도우 크기(한 번에 수신 받고자 하는 크기)

세그먼트는 최대 크기가 MSS(Maximum Segment Size)로 제한되는데, 더 큰 데이터를 전송하고자 할 경우 여러 개의 세그먼트로 나누어 전송하게 된다.  
나뉘어진 세그먼트의 순서를 보장하기 위해 Sequence Number를 통해 순서를 붙이고, Acknowledgement Number를 통해 수신지에서 다음 세그먼트를 수신받길 기대하는 바이트 번호를 전송한다.

![Segment Sequence](image/segment_sequence.png)

### UDP(User Datagram Protocol)

UDP는 TCP와 비교했을 때 기능이 없는 프로토콜로, 단지 IP 패킷을 캡슐화하는 역할만 수행한다.  
때문에 포트 정보와 길이, 체크섬(신뢰성과 관련 없는 필드) 정도의 정보만 가지고 있다.

|            TCP             |          UDP           |
|:--------------------------:|:----------------------:|
| 연결 지향(Connection Oriented) | 비연결 지향(Connectionless) |
|           신뢰성 보장           |        신뢰성 보장 X        |
|           순서 보장            |        순서 보장 X         |
|           흐름 제어            |        흐름 제어 X         |
|           혼잡 제어            |        혼잡 제어 X         |
|           느린 성능            |         빠른 성능          |

### TCP Connection Flow

TCP는 연결을 수립하고, 연결을 끊는 과정을 거치는 연결 지향형 프로토콜로, 신뢰성을 보장하기 위해 아래와 같은 과정을 거친다.

1. 연결 수립
2. 데이터 송수신
3. 연결 해제

전체적으로 위의 과정을 통해 TCP는 신뢰성을 보장하는데, 연결 수립과 해제 과정은 아래와 같다.

- 연결 수립 - Three-way Handshake

1. 호스트 A가 호스트 B에게 SYN 비트가 1로 초기화된 TCP 세그먼트를 전송
2. 호스트 B가 호스트 A에게 SYN 비트와 ACK 비트가 1로 초기화된 TCP 세그먼트를 전송
3. 호스트 A가 호스트 B에게 ACK 비트가 1로 초기화된 TCP 세그먼트를 전송

- 연결 해제 - Four-way Handshake

1. 호스트 A가 호스트 B에게 FIN 비트가 1로 초기화된 TCP 세그먼트를 전송
2. 호스트 B가 호스트 A에게 ACK 비트가 1로 초기화된 TCP 세그먼트를 전송
3. 호스트 B가 호스트 A에게 FIN 비트가 1로 초기화된 TCP 세그먼트를 전송
4. 호스트 A가 호스트 B에게 ACK 비트가 1로 초기화된 TCP 세그먼트를 전송
    - 호스트 B는 ACK를 받은 뒤에 바로 연결 해제
    - 호스트 A는 ACK를 보낸 뒤 일정 시간을 기다린 뒤 연결 해제

연결 수립과 데이터 송수신, 연결 해제 과정에 따라 TCP는 다양한 상태를 가지게 된다.(Stateful)  
이러한 상태를 표현하기 위해 TCP는 상태(state)를 가지고 있다.

- TCP 상태

|      상태      |                          설명                          |
|:------------:|:----------------------------------------------------:|
|    CLOSED    |                    아무런 연결이 없는 상태                     |
|    LISTEN    |      SYN 세그먼트를 기다리는 상태, 보통 서버 호스트가 요청을 기다리는 상태       |
|   SYN-SENT   |         SYN 세그먼트를 보낸 뒤 SYN-ACK 세그먼트를 기다리는 상태         |
| SYN-RECEIVED |         SYN-ACK 세그먼트를 보낸 뒤 ACK 세그먼트를 기다리는 상태         |
| ESTABLISHED  | 연결이 수립된 상태로, Three-way Handshake가 완료되어 데이터를 송수신하는 상태 |

그 외에 FIN-WAIT-1 / FIN-WAIT-2 / CLOSE-WAIT / LAST-ACK / TIME-WAIT 는 연결 해제 과정에서 사용되는 상태이며, 연결이 종료되면 CLOSED 상태가 된다.  
연결 수립 및 해제 과정과 상태 변경에 대한 흐름은 아래 그림과 같다.

![TCP Connection Flow](image/tcp_connection_flow.png)

###### 참고자료

- [현실 세상의 컴퓨터 공학 지식 - 네트워크](https://fastcampus.co.kr/dev_online_newcomputer)