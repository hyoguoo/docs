# 네트워크 계층 (Network Layer) - OSI 3 Layer

## IpV4

인터넷에 연결 된 텀퓨터를 식별하기 위해 부여하는 고유 번호로 32비트로 구성되어 있으며 8비트씩 4개의 옥텟으로 구성되어 있다.

```
192.168.0.10
- 192.168.0: 네트워크 ID
- 10: 호스트 ID
```

위 주소는 네트워크 ID와 호스트 ID로 구성되어 있으며, 해당 ID를 통해 아래와 같은 과정을 거쳐 통신이 이루어진다.

1. 네트워크 ID를 통해 해당 네트워크에 진입
2. 해당 네트워크에 진입한 뒤 호스트 ID를 통해 해당 호스트에 접근

## L3 Packet

일반적으로 네트워크에서의 패킷은 L3 Packet이라고 불리며, 개념적으로 봤을 때 단위 데이터를 의미하며, 논리적 구조로 Header와 Payload로 구성되어 있다.

- Header: 패킷을 전송하기 위한 정보(Source / Destination)
- Payload: 실제 전송할 데이터

패킷의 최대 크기(Header + Payload)는 MTU(Maximum Transmission Unit)라고 불리며, 일반적으로 1500byte이다.

###### 출처

- [외워서 끝내는 네트워크 핵심이론 - 기초](https://www.inflearn.com/course/네트워크-핵심이론-기초)