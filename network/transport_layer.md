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

###### 참고자료

- [현실 세상의 컴퓨터 공학 지식 - 네트워크](https://fastcampus.co.kr/dev_online_newcomputer)