---
layout: editorial
---

# Network Access Layer(네트워크 엑세스 계층) - TCP/IP Layer 1

> LAN 카드와 같은 네트워크에 접속하는 하드웨어적인 면을 다루는 계층, OSI 7계층에서 물리 계층과 데이터 링크 계층과 대응된다.

유/무선 랜 카드, 허브, 리피터 등의 하드웨어 장비이며, 각 장비들이 MAC 주소를 가지고 통신을 한다. 때문에 하나의 컴퓨터에 여러 개의 NIC가 존재할 수 있다.  
물리적인 네트워크로부터 전송된 데이터를 프레임(frame) 단위로 나누고, 에러 검출 및 수정, 흐름 제어 등을 수행한다.

## 이더넷

> 이더넷(Ethernet)은 LAN(Local Area Network)에서 사용하는 현대 유선 LAN에서 가장 많이 사용되는 통신 규약으로, IEEE 802.3 표준을 기반으로 하고, 스펙이 정의되어있다.

이더넷 네트워크에서는 데이터를 프레임(frame) 단위로 나누어 전송하며, 프레임은 아래와 같은 구조로 이루어져 있다.

- 이더넷 프레임(Ethernet Frame)

```
- Preamble: 프레임의 시작을 알리는 비트열
- Destination/Source MAC Address: 프레임의 목적지와 송신지 MAC 주소로, 네트워크 장치(NIC)마다 할당된 고유한 물리적 주소
- EtherType: 프레임의 상위 프로토콜을 명시하는 필드
- Payload: 네트워크 계층으로부터 전달 받은 전송할 데이터
- FCS(Frame Check Sequence): 프레임의 에러 검출을 위한 CRC(Cyclic Redundancy Check) 값이 명시되는 필드
```

## 관련 네트워크 장비

TCP/IP 1계층에서 사용되는 네트워크 장비로는 허브(Hub)와 스위치(Switch)가 있다.

### 허브

물리 계층(OSI Layer 1)의 장비로, 데이터링크 계층(OSI Layer 2)이 아니기 때문에 MAC과 같은 주소 개념이 없어 모든 포트에 연결된 디바이스에 동일한 데이터를 전송한다.  
송신 혹은 수신이 한 번에 한 번만 이루어지는(ex. 무전기) 반이중 통신(half-duplex)만 지원하여, 동시에 여러 디바이스가 데이터를 전송할 경우 충돌(Collision)이 발생한다.

### 스위치

위와 같이 모든 포트에 연결된 디바이스에 전달하고, 반이중 통신만 지원한다는 단점을 해결한 장비이다.
스위치는 데이터링크 계층(OSI Layer 2)에서 동작하는 네트워크 장비로, 목적지 포트로만 데이터를 전송하며, MAC 주소 학습, VLAN 지원, 충돌 없음 등의 특징을 가진다.

#### 스위치의 종류

- L2 Access switch
    - End-Point와 직접 연결되는 스위치
    - Mac 주소를 기반으로 스위칭
    - 대략 사무실 방 단위
- L2 Distribution switch
    - L2 Access switch를 연결하는 스위치
    - VLAN(Virtual LAN) 기능 제공
    - 대략 사무실 층 단위

### 비교

|      특성      |        허브        |          스위치          |
|:------------:|:----------------:|:---------------------:|
|      계층      | OSI 계층 1 (물리 계층) | OSI 계층 2 (데이터 링크 계층)  |
|    주소 할당     |  없음 (MAC 주소 없음)  | MAC 주소 사용하여 특정 포트로 전송 |
|    통신 방식     | 반 이중 (한 번에 한 방향) |     전 이중 (동시 송수신)     |
|    충돌 처리     |    CSMA/CD[1]    |         충돌 없음         |
|    트래픽 관리    |  모든 포트로 브로드캐스트   |   특정 포트로 트래픽 직접 전달    |
| MAC 주소 학습[2] |        X         |           O           |
|  VLAN 지원[3]  |        X         |           O           |

#### 1. CSMA/CD(Carrier Sense Multiple Access with Collision Detection)

반이중 이더넷에서 현재 전송 중인 것이 있는지 확인하고(Carrier Sense),  
여러 장치가 접근해(Multiple Access)  
충돌이 발생한 경우(Collision Detection) 일정 시간 동안 대기한 후 재전송하여 충돌을 해결하는 방식

#### 2. MAC 주소 학습 기능

연결된 디바이스의 MAC 주소를 학습하고, 해당 MAC 주소를 가진 디바이스가 연결된 포트로만 프레임을 전송한다.  
스위치에서의 MAC 주소 학습 기능의 학습과정은 아래와 같다.

1. 플러딩: 허브와 같이 스위치에 연결된 모든 포트로 프레임을 전송
2. 포워딩과 필터링: 수신한 프레임의 목적지 MAC 주소를 참조하여 정확한 포트로 프레임을 전송하고, 해당 MAC 주소를 가진 디바이스가 연결된 포트로만 프레임을 전송
3. 에이징: 일정 시간 동안 프레임을 수신하지 못한 포트의 MAC 주소를 삭제

#### 3. VLAN

가상의 LAN을 만들어 물리적인 네트워크 구조와는 상관없이 논리적인 네트워크 구조를 구성하는 기술로, 스위치를 사용하여 VLAN을 구성할 수 있다.

- 포트 기반 VLAN(정적 VLAN): 물리적 포트에 VLAN을 설정하고 해당 포트에 연결된 디바이스만 특정 VLAN에 속하게 설정
- MAC 주소 기반 VLAN(동적 VLAN): MAC 주소를 기반으로 VLAN을 설정하고, 해당 MAC 주소를 가진 디바이스만 특정 VLAN에 속하게 설정

###### 참고자료

- [외워서 끝내는 네트워크 핵심이론 - 기초](https://www.inflearn.com/course/네트워크-핵심이론-기초)
- [현실 세상의 컴퓨터 공학 지식 - 네트워크](https://fastcampus.co.kr/dev_online_newcomputer)